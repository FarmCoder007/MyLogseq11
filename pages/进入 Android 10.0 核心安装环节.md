title:: 进入 Android 10.0 核心安装环节

- ![image.png](../assets/image_1689156678815_0.png)
- ## processPendingInstall：
	- 代码
		- ```java
		  private void processPendingInstall(final InstallArgs args, final int currentStatus) {
		      if (args.mMultiPackageInstallParams != null) {
		      		args.mMultiPackageInstallParams.tryProcessInstallRequest(args, currentStatus);
		      } else {
		          //1.设置安装参数
		          PackageInstalledInfo res = createPackageInstalledInfo(currentStatus);
		          //2.创建一个新线程，处理安装参数，进行安装
		          processInstallRequestsAsync(
		          res.returnCode == PackageManager.INSTALL_SUCCEEDED,
		          Collections.singletonList(new InstallRequest(args, res)));
		      }
		  }
		  private void processInstallRequestsAsync(boolean success,
		      List<InstallRequest> installRequests) {
		      mHandler.post(() -> {
		          if (success) {
		              for (InstallRequest request : installRequests) {
		                  //1.如果之前安装失败，清除无用信息
		                  request.args.doPreInstall(request.installResult.returnCode);
		              }
		              synchronized (mInstallLock) {
		                  //2. installPackagesTracedLI 是安装过程的核心方法，然后调用
		                  installPackagesLI 进行安装。
		                  // 【同学们注意】下面会分析此函数 installPackagesTracedLI
		                  installPackagesTracedLI(installRequests);
		              }
		              for (InstallRequest request : installRequests) {
		                  //3.如果之前安装失败，清除无用信息
		                  request.args.doPostInstall(
		                  request.installResult.returnCode,
		                  request.installResult.uid);
		              }
		          }
		          for (InstallRequest request : installRequests) {
		              restoreAndPostInstall(request.args.user.getIdentifier(),
		              request.installResult,
		              new PostInstallData(request.args, request.installResult,
		              null));
		          }
		      });
		  }
		  ```
- ## installPackagesTracedLI
	- 代码installPackagesLI **[[#red]]==扫描apk信息==**
		- ```java
		  private void installPackagesLI(List<InstallRequest> requests) {
		      ...
		      // 环节一.Prepare 准备：分析任何当前安装状态，分析包并对其进行初始验证。
		      prepareResult = preparePackageLI(request.args, request.installResult);
		      ...
		      // 环节二.Scan 扫描：考虑到prepare中收集的上下文，询问已分析的包。
		      final List<ScanResult> scanResults = scanPackageTracedLI(
		                                prepareResult.packageToScan,
		                                prepareResult.parseFlags,
		                                prepareResult.scanFlags, System.currentTimeMillis(),
		                                request.args.user);
		      ...
		      // 环节三.Reconcile 调和：在彼此的上下文和当前系统状态中验证扫描的包，以确保安装成功。
		      ReconcileRequest reconcileRequest = new ReconcileRequest(preparedScans,
		                                          installArgs,
		                                          installResults,
		                                          prepareResults,
		                                          mSharedLibraries,
		                                          Collections.unmodifiableMap(mPackages), versionInfos,
		                                          lastStaticSharedLibSettings);
		      ...
		      // 环节四.Commit 提交：提交所有扫描的包并更新系统状态。这是安装流中唯一可以修改系统状态的
		      // 地方，必须在此阶段之前确定所有可预测的错误。
		      commitPackagesLocked(commitRequest);
		      ...
		      // 环节五.完成APK的安装【同学们注意：下面会分析这个操作】
		      executePostCommitSteps(commitRequest);
		  }
		  ```
- ## **executePostCommitSteps**
	- **executePostCommitSteps **安装APK,并为新的代码路径准备应用程序配置文件,并再次检查是否需要**dex****优化**
	- 如果是直接安装新包，会为新的代码路径准备应用程序配置文件
	- 如果是替换安装：其主要过程为更新设置，清除原有的某些APP数据，重新生成相关的app数据目录等步骤，同时要区分系统应用替换和非系统应用替换。而安装新包：则直接更新设置，生成APP数据即可。
	- [PackageManagerService.java] executePostCommitSteps()
		- ```java
		  private void executePostCommitSteps(CommitRequest commitRequest) {
		      for (ReconciledPackage reconciledPkg :commitRequest.reconciledPackages.values()) {
		          ...
		          //1)进行安装
		          prepareAppDataAfterInstallLIF(pkg);
		          //2)如果需要替换安装，则需要清楚原有的APP数据
		          if (reconciledPkg.prepareResult.clearCodeCache) {
		              clearAppDataLIF(pkg, UserHandle.USER_ALL, FLAG_STORAGE_DE |
		              FLAG_STORAGE_CE | FLAG_STORAGE_EXTERNAL | Installer.FLAG_CLEAR_CODE_CACHE_ONLY);
		          }
		          //3)为新的代码路径准备应用程序配置文件。这需要在调用dexopt之前完成，以便任何安装时配
		          // 置文件都可以用于优化。
		          mArtManagerService.prepareAppProfiles(
		                  pkg,resolveUserIds(reconciledPkg.installArgs.user.getIdentifier()),
		                  /* updateReferenceProfileContent= */ true);
		          final boolean performDexopt =
		                          (!instantApp || Global.getInt(mContext.getContentResolver(),
		                          Global.INSTANT_APP_DEXOPT_ENABLED, 0) != 0)
		                          && ((pkg.applicationInfo.flags &
		                          ApplicationInfo.FLAG_DEBUGGABLE) == 0);
		          if (performDexopt) {
		                ...
		              //4)执行dex优化
		              mPackageDexOptimizer.performDexOpt(pkg,null /* instructionSets */,
		                                        getOrCreateCompilerPackageStats(pkg),
		                                        mDexManager.getPackageUseInfoOrDefault(packageName),
		                                        dexoptOptions);
		          }
		          BackgroundDexOptService.notifyPackageChanged(packageName);
		      }
		  }
		  ```
- ## prepareAppDataAfterInstallLIF:
	- 代码
		- ```java
		  通过一系列的调用，最终会调用到[Installer.java] createAppData()进行安装，交给installed进程
		  进行APK的安装
		  调用栈如下：
		  prepareAppDataAfterInstallLIF()
		  |
		  prepareAppDataLIF()
		  |
		  prepareAppDataLeafLIF()
		  |
		  [Installer.java]
		  createAppData()
		  
		  private void prepareAppDataAfterInstallLIF(PackageParser.Package pkg) {
		  ...
		  for (UserInfo user : um.getUsers()) {
		  ...
		  if (ps.getInstalled(user.id)) {
		  // TODO: when user data is locked, mark that we're still dirty
		  prepareAppDataLIF(pkg, user.id, flags);
		  }
		  }
		  }
		  private void prepareAppDataLIF(PackageParser.Package pkg, int userId, int flags)
		  {
		        if (pkg == null) {
		            Slog.wtf(TAG, "Package was null!", new Throwable());
		            return;
		        }
		        prepareAppDataLeafLIF(pkg, userId, flags);
		        final int childCount = (pkg.childPackages != null) ?
		        pkg.childPackages.size() : 0;
		        for (int i = 0; i < childCount; i++) {
		            prepareAppDataLeafLIF(pkg.childPackages.get(i), userId, flags);
		        }
		  }
		  
		  private void prepareAppDataLeafLIF(PackageParser.Package pkg, int userId, int flags) {
		      ...
		      try {
		        // 调用Installd守护进程的入口
		        ceDataInode = mInstaller.createAppData(volumeUuid, packageName, userId,
		        flags,appId, seInfo, app.targetSdkVersion);
		      } catch (InstallerException e) {
		          if (app.isSystemApp()) {
		         		 destroyAppDataLeafLIF(pkg, userId, flags);
		                try {
		                ceDataInode = mInstaller.createAppData(volumeUuid, packageName,
		                                        userId, flags,
		                                        appId, seInfo, app.targetSdkVersion);
		                } catch (InstallerException e2) {
		                  ...
		                }
		      	}
		      }
		  }
		  ```
- ## Installer.**createAppData **收尾工作，安装完成后，更新设置，更新安装锁等：
	- [Installer.java]
		- ```java
		  public long createAppData(String uuid, String packageName, int userId, int
		                flags, int appId,
		                String seInfo, int targetSdkVersion) throws InstallerException {
		      if (!checkBeforeRemote()) return -1;
		      try {
		      // mInstalld 为IInstalld的对象，即通过Binder调用到 进程installd，最终调用
		      // installd的createAppData()
		      // 【同学们注意】 mInstalld是一个aidl文件，通过此aidl文件调用到 Binder机制的服务
		      // 端，服务端哪里要操控底层....
		      return mInstalld.createAppData(uuid, packageName, userId, flags, appId,
		      										seInfo,targetSdkVersion);
		      } catch (Exception e) {
		      		throw InstallerException.from(e);
		      }
		  }
		  ```