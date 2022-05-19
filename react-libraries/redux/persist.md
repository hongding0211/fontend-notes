# 如何持久化 Redux 数据

## redux-persist

改写 **store.js** 文件

```js
import {combineReducers, configureStore} from "@reduxjs/toolkit";
import persistReducer from "redux-persist/es/persistReducer";
import storage from 'redux-persist/lib/storage'
import {FLUSH, PAUSE, PERSIST, PURGE, REGISTER, REHYDRATE} from "redux-persist/es/constants";

export default configureStore({
    reducer: persistReducer(
        {key: 'root', storage},
        combineReducers({
           xxx: xxxReducer...
        })
    ),
  	// 需要添加下列字段，不然会报错
    middleware: getDefaultMiddleware => getDefaultMiddleware({
        serializableCheck: {
            ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
        }
    }),
})
```

然后在 `<Provider store={store}>` 标签内再嵌套

```js
let persistor = persistStore(store)

<Provider store={store}>
  <PersistGate loading={null} persistor={persistor}>
  </PersistGate>
</Provider>
```
