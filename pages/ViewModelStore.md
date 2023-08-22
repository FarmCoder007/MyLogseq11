- 源码,Map存储viewModel、、、String  作为Key
  collapsed:: true
	- ```java
	  public class ViewModelStore {
	  
	      private final HashMap<String, ViewModel> mMap = new HashMap<>();
	  
	      final void put(String key, ViewModel viewModel) {
	          ViewModel oldViewModel = mMap.put(key, viewModel);
	          if (oldViewModel != null) {
	              oldViewModel.onCleared();
	          }
	      }
	  
	      final ViewModel get(String key) {
	          return mMap.get(key);
	      }
	  
	      Set<String> keys() {
	          return new HashSet<>(mMap.keySet());
	      }
	  
	      /**
	       *  Clears internal storage and notifies ViewModels that they are no longer used.
	       */
	      public final void clear() {
	          for (ViewModel vm : mMap.values()) {
	              vm.clear();
	          }
	          mMap.clear();
	      }
	  }
	  
	  ```