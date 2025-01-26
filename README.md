**Cheatsheet detallado** sobre **Redux Toolkit** (RTK) en combinación con **React**. Redux Toolkit es la forma recomendada de trabajar con Redux hoy en día, ya que simplifica mucho la configuración y el manejo del estado global.

---

## **1. Instalación**
```bash
npm install @reduxjs/toolkit react-redux
```

---

## **2. Configuración Básica**

### **Crear un Store**
El store es donde se almacena el estado global de la aplicación.

```javascript
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer, // Agrega tus reducers aquí
  },
});
```

### **Proveer el Store a la Aplicación**
Envuelve tu aplicación con el `Provider` de React-Redux.

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { store } from './app/store';
import App from './App';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

---

## **3. Crear un Slice**
Un **slice** es una porción del estado global que incluye el reducer y las acciones.

```javascript
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter', // Nombre del slice
  initialState: { value: 0 }, // Estado inicial
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

// Exportar las acciones
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// Exportar el reducer
export default counterSlice.reducer;
```

---

## **4. Usar el Estado y las Acciones en Componentes**

### **Leer el Estado con `useSelector`**
```javascript
import React from 'react';
import { useSelector } from 'react-redux';

const CounterComponent = () => {
  const count = useSelector((state) => state.counter.value);

  return <div>{count}</div>;
};
```

### **Despachar Acciones con `useDispatch`**
```javascript
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { increment, decrement } from './features/counterSlice';

const CounterComponent = () => {
  const dispatch = useDispatch();
  const count = useSelector((state) => state.counter.value);

  return (
    <div>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
    </div>
  );
};
```

---

## **5. Acciones Asíncronas con `createAsyncThunk`**
Para manejar operaciones asíncronas (como llamadas a APIs), usa `createAsyncThunk`.

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import axios from 'axios';

// Crear un thunk para una operación asíncrona
export const fetchUserData = createAsyncThunk(
  'users/fetchUserData',
  async (userId) => {
    const response = await axios.get(`https://api.example.com/users/${userId}`);
    return response.data;
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: { data: null, status: 'idle', error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserData.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUserData.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchUserData.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

export default usersSlice.reducer;
```

---

## **6. Acceder al Estado Asíncrono en Componentes**
```javascript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUserData } from './features/usersSlice';

const UserComponent = ({ userId }) => {
  const dispatch = useDispatch();
  const { data, status, error } = useSelector((state) => state.users);

  useEffect(() => {
    dispatch(fetchUserData(userId));
  }, [dispatch, userId]);

  if (status === 'loading') return <div>Loading...</div>;
  if (status === 'failed') return <div>Error: {error}</div>;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  );
};
```

---

## **7. Crear un Selector con `createSelector`**
Para seleccionar y derivar datos del estado de manera eficiente.

```javascript
import { createSelector } from '@reduxjs/toolkit';

const selectCounter = (state) => state.counter;

export const selectCounterValue = createSelector(
  [selectCounter],
  (counter) => counter.value
);

// Uso en un componente
const count = useSelector(selectCounterValue);
```

---

## **8. Middleware y Configuraciones Adicionales**
Puedes agregar middlewares como `redux-logger` o `redux-thunk` (ya incluido en RTK).

```javascript
import { configureStore, getDefaultMiddleware } from '@reduxjs/toolkit';
import logger from 'redux-logger';
import rootReducer from './reducers';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger),
});
```

---

## **9. Resumen de Conceptos Clave**
- **Store**: Contenedor del estado global.
- **Slice**: Porción del estado con reducers y acciones.
- **Reducer**: Función que define cómo cambia el estado.
- **Action**: Objeto que describe un cambio en el estado.
- **Dispatch**: Función para enviar acciones al store.
- **Selector**: Función para leer datos del estado.
- **Thunk**: Middleware para manejar operaciones asíncronas.

---

## **10. Ejemplo Completo**
Aquí tienes un ejemplo completo de una aplicación de contador:

```javascript
// features/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;

// app/store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

// App.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from './features/counterSlice';

const App = () => {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
    </div>
  );
};

export default App;
```

---

Este cheatsheet cubre los conceptos esenciales de **Redux Toolkit** en combinación con **React**. ¡Espero que te sea útil! 🚀
