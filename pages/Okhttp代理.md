- 在使用OkHttp时，如果用户在创建 OkHttpClient 时，配置了 proxy 或者 proxySelector ，则会使用配置的代理，并且 proxy 优先级高于 proxySelector 。而如果未配置，则会获取机器配置的代理并使用。
	- ```java
	   //JDK : ProxySelector
	      try {
	          URI uri = new URI("http://restapi.amap.com");
	          List<Proxy> proxyList = ProxySelector.getDefault().select(uri);
	          System.out.println(proxyList.get(0).address());
	          System.out.println(proxyList.get(0).type());
	      } catch (URISyntaxException e) {
	          e.printStackTrace();
	      }
	  ```
- 因此，如果我们不需要自己的App中的请求走代理，则可以配置一个 proxy(Proxy.NO_PROXY) ，这样也可以避免被抓包。 NO_PROXY 的定义如下
	- ```java
	  public static final Proxy NO_PROXY = new Proxy();
	  private Proxy() {
	          this.type = Proxy.Type.DIRECT;
	          this.sa = null;
	      }
	  }
	  ```
- 代理在Java中对应的抽象类有三种类型:
	- ```java
	   public static enum Type {
	          DIRECT,
	          HTTP,
	          SOCKS;
	          private Type() {
	          }
	      }
	  ```
- DIRECT ：无代理， HTTP ：http代理， SOCKS ：socks代理。第一种自然不用多说，而Http代理与Socks代理有什么区别？
- 对于Socks代理，在HTTP的场景下，代理服务器完成TCP数据包的转发工作; 而Http代理服务器，在转发数据之外，还会解析HTTP的请求及响应，并根据请求及响应的内容做一些处理。
- RealConnection 的 connectSocket 方法
	- ```java
	  //如果是Socks代理则 new Socket(proxy); 否则无代理或http代理就
	      address.socketFactory().createSocket()，相当于直接:new Socket()
	      rawSocket = proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.HTTP
	              ? address.socketFactory().createSocket()
	              : new Socket(proxy);
	  //connect方法
	      socket.connect(address);
	  ```
- 设置了SOCKS代理的情况下，创建Socket时，为其传入proxy，写代码时连接时还是以HTTP服务器为目标地址（实际上Socket肯定是与SOCKS代理服务器连）；但是如果设置的是Http代理，创建的Socket是与Http代理服务器建立连接。
- >在 connect 方法时传递的 address 来自于下面的集合 inetSocketAddresses RouteSelector 的resetNextInetSocketAddress 方法：
- ```java
      private void resetNextInetSocketAddress(Proxy proxy) throws IOException {
  // ......
          if (proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.SOCKS) {
  //无代理和socks代理，使用http服务器域名与端口
              socketHost = address.url().host();
              socketPort = address.url().port();
          } else {
              SocketAddress proxyAddress = proxy.address();
              if (!(proxyAddress instanceof InetSocketAddress)) {
                  throw new IllegalArgumentException(
                          "Proxy.address() is not an " + "InetSocketAddress: " + proxyAddress.getClass());
              }
              InetSocketAddress proxySocketAddress = (InetSocketAddress) proxyAddress;
              socketHost = getHostString(proxySocketAddress);
              socketPort = proxySocketAddress.getPort();
          }
  // ......
          if (proxy.type() == Proxy.Type.SOCKS) {
  //socks代理 connect http服务器 （DNS没用，由代理服务器解析域名）
              inetSocketAddresses.add(InetSocketAddress.createUnresolved(socketHost, socketPort));
          } else {
  //无代理，dns解析http服务器
  //http代理,dns解析http代理服务器
              List<InetAddress> addresses = address.dns().lookup(socketHost);
  //......
              for (int i = 0, size = addresses.size(); i < size; i++) {
                  InetAddress inetAddress = addresses.get(i);
                  inetSocketAddresses.add(new InetSocketAddress(inetAddress, socketPort));
              }
          }
      }
  ```
- 设置代理时，Http服务器的域名解析会被交给代理服务器执行。但是如果是设置了Http代理，会对Http代理服务器的域名使用 OkhttpClient 配置的dns解析代理服务器，Http服务器的域名解析被交给代理服务器解析。
-
- 上述代码就是代理与DNS在OkHttp中的使用，但是还有一点需要注意，Http代理也分成两种类型：普通代理与隧道代理。
-
- 其中普通代理不需要额外的操作，扮演「中间人」的角色，在两端之间来回传递报文。这个“中间人”在收到客户端发送的请求报文时，需要正确的处理请求和连接状态，同时向服务器发送新的请求，在收到响应后，将响应结果包装成一个响应体返回给客户端。在普通代理的流程中，代理两端都是有可能察觉不到"中间人“的存在。
-
- 但是隧道代理不再作为中间人，无法改写客户端的请求，而仅仅是在建立连接后，将客户端的请求，通过建立好的隧道，无脑的转发给终端服务器。隧道代理需要发起Http **CONNECT**请求，这种请求方式没有请求体，仅供代理服务器使用，并不会传递给终端服务器。请求头 部分一旦结束，后面的所有数据，都被视为应该转发给终端服务器的数据，代理需要把他们无脑的直接转发，直到从客户端的 TCP 读通道关闭。**CONNECT **的响应报文，在代理服务器和终端服务器建立连接后，可以向客户端返回一个 200 Connect established 的状态码，以此表示和终端服务器的连接，建立成功。
- >RealConnection的connect方法
	- ```java
	  
	      if (route.requiresTunnel()) {
	          connectTunnel(connectTimeout, readTimeout, writeTimeout, call, eventListener);
	          if (rawSocket == null) {
	  // We were unable to connect the tunnel but properly closed down our
	  // resources.
	              break;
	          }
	      } else {
	          connectSocket(connectTimeout, readTimeout, call, eventListener);
	      }
	  ```
- requiresTunnel 方法的判定为：当前请求为https并且存在http代理，这时候 connectTunnel 中会发起:
	- ```java
	  CONNECT xxxx HTTP/1.1
	  Host: xxxx
	  Proxy-Connection: Keep-Alive
	  User-Agent: okhttp/${version}
	  ```
- 的请求，连接成功代理服务器会返回200；如果返回407表示代理服务器需要鉴权(如：付费代理)，这时需要在请求头中加入 Proxy-Authorization ：
	- ```java
	    Authenticator authenticator = new Authenticator() {
	          @Nullable
	          @Override
	          public Request authenticate(Route route, Response response) throws IOException {
	                if(response.code == 407){
	                           //代理鉴权
	                      String credential = Credentials.basic("代理服务用户名", "代理服务密码");
	                      return response.request().newBuilder()
	                              .header("Proxy-Authorization", credential)
	                              .build();
	              }
	              return null;
	          }
	      };
	      new OkHttpClient.Builder().proxyAuthenticator(authenticator);
	  ```