## 背景
collapsed:: true
	- 在构建获取ServiceMethod时，调用了ServiceMethod.parseAnnotations，先解析了注解
-
- ## RequestFactory.parseAnnotations，创建RequestFactory实例
	- ```java
	    static RequestFactory parseAnnotations(Retrofit retrofit, Method method) {
	      return new Builder(retrofit, method).build();
	    }
	  ```
- ## 看build()函数：分析方法和参数的注解
	- ```java
	  RequestFactory build() {
	        for (Annotation annotation : methodAnnotations) {
	          // 核心方法1，分析方法注解
	          parseMethodAnnotation(annotation);
	        }
	  
	        if (httpMethod == null) {
	          throw methodError(method, "HTTP method annotation is required (e.g., @GET, @POST, etc.).");
	        }
	  
	        if (!hasBody) {
	          if (isMultipart) {
	            throw methodError(
	                method,
	                "Multipart can only be specified on HTTP methods with request body (e.g., @POST).");
	          }
	          if (isFormEncoded) {
	            throw methodError(
	                method,
	                "FormUrlEncoded can only be specified on HTTP methods with "
	                    + "request body (e.g., @POST).");
	          }
	        }
	        // 分析参数注解 
	        int parameterCount = parameterAnnotationsArray.length;
	        parameterHandlers = new ParameterHandler<?>[parameterCount];
	        for (int p = 0, lastParameter = parameterCount - 1; p < parameterCount; p++) {
	          parameterHandlers[p] =
	              parseParameter(p, parameterTypes[p], parameterAnnotationsArray[p], p == lastParameter);
	        }
	  
	        if (relativeUrl == null && !gotUrl) {
	          throw methodError(method, "Missing either @%s URL or @Url parameter.", httpMethod);
	        }
	        if (!isFormEncoded && !isMultipart && !hasBody && gotBody) {
	          throw methodError(method, "Non-body HTTP method cannot contain @Body.");
	        }
	        if (isFormEncoded && !gotField) {
	          throw methodError(method, "Form-encoded method must contain at least one @Field.");
	        }
	        if (isMultipart && !gotPart) {
	          throw methodError(method, "Multipart method must contain at least one @Part.");
	        }
	  
	        return new RequestFactory(this);
	      }
	  ```
- ## 核心方法1：parseMethodAnnotation:分析方法的注解
	- ```java
	  private void parseMethodAnnotation(Annotation annotation) {
	        if (annotation instanceof DELETE) {
	          parseHttpMethodAndPath("DELETE", ((DELETE) annotation).value(), false);
	        } else if (annotation instanceof GET) {
	          parseHttpMethodAndPath("GET", ((GET) annotation).value(), false);
	        } else if (annotation instanceof HEAD) {
	          parseHttpMethodAndPath("HEAD", ((HEAD) annotation).value(), false);
	        } else if (annotation instanceof PATCH) {
	          parseHttpMethodAndPath("PATCH", ((PATCH) annotation).value(), true);
	        } else if (annotation instanceof POST) {
	          parseHttpMethodAndPath("POST", ((POST) annotation).value(), true);
	        } else if (annotation instanceof PUT) {
	          parseHttpMethodAndPath("PUT", ((PUT) annotation).value(), true);
	        } else if (annotation instanceof OPTIONS) {
	          parseHttpMethodAndPath("OPTIONS", ((OPTIONS) annotation).value(), false);
	        } else if (annotation instanceof HTTP) {
	          HTTP http = (HTTP) annotation;
	          parseHttpMethodAndPath(http.method(), http.path(), http.hasBody());
	        } else if (annotation instanceof retrofit2.http.Headers) {
	          String[] headersToParse = ((retrofit2.http.Headers) annotation).value();
	          if (headersToParse.length == 0) {
	            throw methodError(method, "@Headers annotation is empty.");
	          }
	          headers = parseHeaders(headersToParse);
	        } else if (annotation instanceof Multipart) {
	          if (isFormEncoded) {
	            throw methodError(method, "Only one encoding annotation is allowed.");
	          }
	          isMultipart = true;
	        } else if (annotation instanceof FormUrlEncoded) {
	          if (isMultipart) {
	            throw methodError(method, "Only one encoding annotation is allowed.");
	          }
	          isFormEncoded = true;
	        }
	      }
	  ```
	- ## parseHttpMethodAndPath 以get为例具体解析
		- ```java
		  private void parseHttpMethodAndPath(String httpMethod, String value, boolean hasBody) {
		        if (this.httpMethod != null) {
		          throw methodError(method, "Only one HTTP method is allowed. Found: %s and %s.",
		              this.httpMethod, httpMethod);
		        }
		        // 会设置 Get方法
		        this.httpMethod = httpMethod;
		        // 有没有body
		        this.hasBody = hasBody;
		   
		        if (value.isEmpty()) {
		          return;
		        }
		   
		        // Get the relative URL path and existing query string, if present.
		        int question = value.indexOf('?');
		        if (question != -1 && question < value.length() - 1) {
		          // Ensure the query string does not have any named parameters.
		          String queryParams = value.substring(question + 1);
		          Matcher queryParamMatcher = PARAM_URL_REGEX.matcher(queryParams);
		          if (queryParamMatcher.find()) {
		            throw methodError(method, "URL query string \"%s\" must not have replace block. "
		                + "For dynamic query parameters use @Query.", queryParams);
		          }
		        }
		   
		        this.relativeUrl = value;
		        this.relativeUrlParamNames = parsePathParameters(value);
		      }
		  ```
- ## 核心方法2：解析参数注解parseParameter
	- ```java
	   private @Nullable ParameterHandler<?> parseParameter(
	          int p, Type parameterType, @Nullable Annotation[] annotations, boolean allowContinuation) {
	        ParameterHandler<?> result = null;
	        if (annotations != null) {
	          for (Annotation annotation : annotations) {
	            ParameterHandler<?> annotationAction =
	                parseParameterAnnotation(p, parameterType, annotations, annotation);
	  
	            if (annotationAction == null) {
	              continue;
	            }
	  
	            if (result != null) {
	              throw parameterError(
	                  method, p, "Multiple Retrofit annotations found, only one allowed.");
	            }
	  
	            result = annotationAction;
	          }
	        }
	  
	        if (result == null) {
	          if (allowContinuation) {
	            try {
	              if (Utils.getRawType(parameterType) == Continuation.class) {
	                isKotlinSuspendFunction = true;
	                return null;
	              }
	            } catch (NoClassDefFoundError ignored) {
	            }
	          }
	          throw parameterError(method, p, "No Retrofit annotation found.");
	        }
	  
	        return result;
	      }
	  ```
	- 具体解析方法见parseParameterAnnotation
- # [[#red]]==**总结**==
	- [[#red]]==**对我们写的网络请求接口里发方法 做分析 记录   分析 所有的注解  方法   然后分析出完整的拼装方案存在 RequestFactory这里**==