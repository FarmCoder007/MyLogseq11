- build.gradle 中 给kotlin文件单独配置文件路径
	- ```
	      sourceSets {
	          main {
	              manifest.srcFile 'AndroidManifest.xml'
	              java.srcDirs = ['src', 'kotlin']
	              resources.srcDirs = ['src']
	              aidl.srcDirs = ['src']
	              renderscript.srcDirs = ['src']
	              res.srcDirs = [
	                      'res',
	                      'res/specialchannel'
	              ]
	              assets.srcDirs = ['assets']
	              jniLibs.srcDirs = ['libs']
	          }
	  
	          // Move the tests to tests/java, tests/res, etc...
	          androidTest.setRoot('tests')
	  
	          // Move the build types to build-types/<type>
	          // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
	          // This moves them out of them default location under src/<type>/... which would
	          // conflict with src/ being used by the main source set.
	          // Adding new build types or product flavors should be accompanied
	          // by a similar customization.
	          debug.setRoot('build-types/debug')
	          release.setRoot('build-types/release')
	      }
	  
	  
	  ```
-