title:: IPersonManager.java

- ```java
  /*
   * This file is auto-generated.  DO NOT MODIFY.
   */
  public interface IPersonManager extends android.os.IInterface
  {
    /** Default implementation for IPersonManager. */
    public static class Default implements IPersonManager
    {
      @Override public void addPerson(com.lemonydbook.bean.Personon person) throws android.os.RemoteException
      {
      }
      @Override public void addTest(com.lemonydbook.bean.Test test) throws android.os.RemoteException
      {
      }
      @Override public java.util.List<com.lemonydbook.bean.Personon> getPersonList() throws android.os.RemoteException
      {
        return null;
      }
      @Override
      public android.os.IBinder asBinder() {
        return null;
      }
    }
    /** Local-side IPC implementation stub class. */
    public static abstract class Stub extends android.os.Binder implements IPersonManager
    {
      private static final java.lang.String DESCRIPTOR = "IPersonManager";
      /** Construct the stub at attach it to the interface. */
      public Stub()
      {
        this.attachInterface(this, DESCRIPTOR);
      }
      /**
       * Cast an IBinder object into an IPersonManager interface,
       * generating a proxy if needed.
       */
      public static IPersonManager asInterface(android.os.IBinder obj)
      {
        if ((obj==null)) {
          return null;
        }
        android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
        if (((iin!=null)&&(iin instanceof IPersonManager))) {
          return ((IPersonManager)iin);
        }
        return new IPersonManager.Stub.Proxy(obj);
      }
      @Override public android.os.IBinder asBinder()
      {
        return this;
      }
      @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
      {
        java.lang.String descriptor = DESCRIPTOR;
        switch (code)
        {
          case INTERFACE_TRANSACTION:
          {
            reply.writeString(descriptor);
            return true;
          }
          case TRANSACTION_addPerson:
          {
            data.enforceInterface(descriptor);
            com.lemonydbook.bean.Personon _arg0;
            if ((0!=data.readInt())) {
              _arg0 = com.lemonydbook.bean.Personon.CREATOR.createFromParcel(data);
            }
            else {
              _arg0 = null;
            }
            this.addPerson(_arg0);
            reply.writeNoException();
            return true;
          }
          case TRANSACTION_addTest:
          {
            data.enforceInterface(descriptor);
            com.lemonydbook.bean.Test _arg0;
            if ((0!=data.readInt())) {
              _arg0 = com.lemonydbook.bean.Test.CREATOR.createFromParcel(data);
            }
            else {
              _arg0 = null;
            }
            this.addTest(_arg0);
            reply.writeNoException();
            return true;
          }
          case TRANSACTION_getPersonList:
          {
            data.enforceInterface(descriptor);
            java.util.List<com.lemonydbook.bean.Personon> _result = this.getPersonList();
            reply.writeNoException();
            reply.writeTypedList(_result);
            return true;
          }
          default:
          {
            return super.onTransact(code, data, reply, flags);
          }
        }
      }
      private static class Proxy implements IPersonManager
      {
        private android.os.IBinder mRemote;
        Proxy(android.os.IBinder remote)
        {
          mRemote = remote;
        }
        @Override public android.os.IBinder asBinder()
        {
          return mRemote;
        }
        public java.lang.String getInterfaceDescriptor()
        {
          return DESCRIPTOR;
        }
        @Override public void addPerson(com.lemonydbook.bean.Personon person) throws android.os.RemoteException
        {
          android.os.Parcel _data = android.os.Parcel.obtain();
          android.os.Parcel _reply = android.os.Parcel.obtain();
          try {
            _data.writeInterfaceToken(DESCRIPTOR);
            if ((person!=null)) {
              _data.writeInt(1);
              person.writeToParcel(_data, 0);
            }
            else {
              _data.writeInt(0);
            }
            boolean _status = mRemote.transact(Stub.TRANSACTION_addPerson, _data, _reply, 0);
            if (!_status && getDefaultImpl() != null) {
              getDefaultImpl().addPerson(person);
              return;
            }
            _reply.readException();
          }
          finally {
            _reply.recycle();
            _data.recycle();
          }
        }
        @Override public void addTest(com.lemonydbook.bean.Test test) throws android.os.RemoteException
        {
          android.os.Parcel _data = android.os.Parcel.obtain();
          android.os.Parcel _reply = android.os.Parcel.obtain();
          try {
            _data.writeInterfaceToken(DESCRIPTOR);
            if ((test!=null)) {
              _data.writeInt(1);
              test.writeToParcel(_data, 0);
            }
            else {
              _data.writeInt(0);
            }
            boolean _status = mRemote.transact(Stub.TRANSACTION_addTest, _data, _reply, 0);
            if (!_status && getDefaultImpl() != null) {
              getDefaultImpl().addTest(test);
              return;
            }
            _reply.readException();
          }
          finally {
            _reply.recycle();
            _data.recycle();
          }
        }
        @Override public java.util.List<com.lemonydbook.bean.Personon> getPersonList() throws android.os.RemoteException
        {
          android.os.Parcel _data = android.os.Parcel.obtain();
          android.os.Parcel _reply = android.os.Parcel.obtain();
          java.util.List<com.lemonydbook.bean.Personon> _result;
          try {
            _data.writeInterfaceToken(DESCRIPTOR);
            boolean _status = mRemote.transact(Stub.TRANSACTION_getPersonList, _data, _reply, 0);
            if (!_status && getDefaultImpl() != null) {
              return getDefaultImpl().getPersonList();
            }
            _reply.readException();
            _result = _reply.createTypedArrayList(com.lemonydbook.bean.Personon.CREATOR);
          }
          finally {
            _reply.recycle();
            _data.recycle();
          }
          return _result;
        }
        public static IPersonManager sDefaultImpl;
      }
      static final int TRANSACTION_addPerson = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
      static final int TRANSACTION_addTest = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
      static final int TRANSACTION_getPersonList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
      public static boolean setDefaultImpl(IPersonManager impl) {
        if (Stub.Proxy.sDefaultImpl == null && impl != null) {
          Stub.Proxy.sDefaultImpl = impl;
          return true;
        }
        return false;
      }
      public static IPersonManager getDefaultImpl() {
        return Stub.Proxy.sDefaultImpl;
      }
    }
    public void addPerson(com.lemonydbook.bean.Personon person) throws android.os.RemoteException;
    public void addTest(com.lemonydbook.bean.Test test) throws android.os.RemoteException;
    public java.util.List<com.lemonydbook.bean.Personon> getPersonList() throws android.os.RemoteException;
  }
  
  ```