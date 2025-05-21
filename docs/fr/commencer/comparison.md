---
title: Comparaison
description: Comment Zustand se compare aux bibliothèques similaires
nav: 1
id: /1kV230U9XCsl3TClqjDXDLYz0ELD1rSQ1pF9biAITw=
---

Zustand est l'une des nombreuses bibliothèques de gestion d'état pour React.
Sur cette page, nous discuterons de Zustand
en comparaison avec certaines de ces bibliothèques,
notamment Redux, Valtio, Jotai et Recoil.

Chaque bibliothèque a ses propres forces et faiblesses,
et nous comparerons les principales différences et similitudes entre chacune d'elles.

## Redux

### Modèle d'état (vs Redux)

Conceptuellement, Zustand et Redux sont assez similaires,
tous deux sont basés sur un modèle d'état immuable.
Cependant, Redux nécessite que votre application soit enveloppée
dans des fournisseurs de contexte ; Zustand non.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  increment: (qty: number) => void
  decrement: (qty: number) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  increment: (qty: number) => set((state) => ({ count: state.count + qty })),
  decrement: (qty: number) => set((state) => ({ count: state.count - qty })),
}))
```

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  increment: (qty: number) => void
  decrement: (qty: number) => void
}

type Action = {
  type: keyof Actions
  qty: number
}

const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  dispatch: (action: Action) => set((state) => countReducer(state, action)),
}))
```

**Redux**

```ts
import { createStore } from 'redux'
import { useSelector, useDispatch } from 'react-redux'

type State = {
  count: number
}

type Action = {
  type: 'increment' | 'decrement'
  qty: number
}

const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const countStore = createStore(countReducer)
```

```ts
import { createSlice, configureStore } from '@reduxjs/toolkit'

const countSlice = createSlice({
  name: 'count',
  initialState: { value: 0 },
  reducers: {
    incremented: (state, qty: number) => {
      // Redux Toolkit ne modifie pas l'état, il utilise la bibliothèque Immer
      // en coulisses, nous permettant d'avoir ce qu'on appelle un "état brouillon".
      state.value += qty
    },
    decremented: (state, qty: number) => {
      state.value -= qty
    },
  },
})

const countStore = configureStore({ reducer: countSlice.reducer })
```

### Optimisation du rendu (vs Redux)

En ce qui concerne les optimisations de rendu dans votre application,
il n'y a pas de différences majeures d'approche entre Zustand et Redux.
Dans les deux bibliothèques, il est recommandé
d'appliquer manuellement des optimisations de rendu en utilisant des sélecteurs.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  increment: (qty: number) => void
  decrement: (qty: number) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  increment: (qty: number) => set((state) => ({ count: state.count + qty })),
  decrement: (qty: number) => set((state) => ({ count: state.count - qty })),
}))

const Component = () => {
  const count = useCountStore((state) => state.count)
  const increment = useCountStore((state) => state.increment)
  const decrement = useCountStore((state) => state.decrement)
  // ...
}
```

**Redux**

```ts
import { createStore } from 'redux'
import { useSelector, useDispatch } from 'react-redux'

type State = {
  count: number
}

type Action = {
  type: 'increment' | 'decrement'
  qty: number
}

const countReducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.qty }
    case 'decrement':
      return { count: state.count - action.qty }
    default:
      return state
  }
}

const countStore = createStore(countReducer)

const Component = () => {
  const count = useSelector((state) => state.count)
  const dispatch = useDispatch()
  // ...
}
```

```ts
import { useSelector } from 'react-redux'
import type { TypedUseSelectorHook } from 'react-redux'
import { createSlice, configureStore } from '@reduxjs/toolkit'

const countSlice = createSlice({
  name: 'count',
  initialState: { value: 0 },
  reducers: {
    incremented: (state, qty: number) => {
      // Redux Toolkit ne modifie pas l'état, il utilise la bibliothèque Immer
      // en coulisses, nous permettant d'avoir ce qu'on appelle un "état brouillon".
      state.value += qty
    },
    decremented: (state, qty: number) => {
      state.value -= qty
    },
  },
})

const countStore = configureStore({ reducer: countSlice.reducer })

const useAppSelector: TypedUseSelectorHook<typeof countStore.getState> =
  useSelector

const useAppDispatch: () => typeof countStore.dispatch = useDispatch

const Component = () => {
  const count = useAppSelector((state) => state.count.value)
  const dispatch = useAppDispatch()
  // ...
}
```

## Valtio

### Modèle d'état (vs Valtio)

Zustand et Valtio abordent la gestion d'état
d'une manière fondamentalement différente.
Zustand est basé sur le modèle d'état **immuable**,
tandis que Valtio est basé sur le modèle d'état **mutable**.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  obj: { count: number }
}

const store = create<State>(() => ({ obj: { count: 0 } }))

store.setState((prev) => ({ obj: { count: prev.obj.count + 1 } }))
```

**Valtio**

```ts
import { proxy } from 'valtio'

const state = proxy({ obj: { count: 0 } })

state.obj.count += 1
```

### Optimisation du rendu (vs Valtio)

L'autre différence entre Zustand et Valtio
est que Valtio effectue des optimisations de rendu grâce à l'accès aux propriétés.
Cependant, avec Zustand, il est recommandé que
vous appliquiez manuellement des optimisations de rendu en utilisant des sélecteurs.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

const useCountStore = create<State>(() => ({
  count: 0,
}))

const Component = () => {
  const count = useCountStore((state) => state.count)
  // ...
}
```

**Valtio**

```ts
import { proxy, useSnapshot } from 'valtio'

const state = proxy({
  count: 0,
})

const Component = () => {
  const { count } = useSnapshot(state)
  // ...
}
```

## Jotai

### Modèle d'état (vs Jotai)

Il y a une différence majeure entre Zustand et Jotai.
Zustand est un magasin unique,
tandis que Jotai est composé d'atomes primitifs qui peuvent être composés ensemble.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  updateCount: (
    countCallback: (count: State['count']) => State['count'],
  ) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  updateCount: (countCallback) =>
    set((state) => ({ count: countCallback(state.count) })),
}))
```

**Jotai**

```ts
import { atom } from 'jotai'

const countAtom = atom<number>(0)
```

### Optimisation du rendu (vs Jotai)

Jotai réalise des optimisations de rendu grâce à la dépendance des atomes.
Cependant, avec Zustand, il est recommandé que
vous appliquiez manuellement des optimisations de rendu en utilisant des sélecteurs.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  updateCount: (
    countCallback: (count: State['count']) => State['count'],
  ) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  updateCount: (countCallback) =>
    set((state) => ({ count: countCallback(state.count) })),
}))

const Component = () => {
  const count = useCountStore((state) => state.count)
  const updateCount = useCountStore((state) => state.updateCount)
  // ...
}
```

**Jotai**

```ts
import { atom, useAtom } from 'jotai'

const countAtom = atom<number>(0)

const Component = () => {
  const [count, updateCount] = useAtom(countAtom)
  // ...
}
```

## Recoil

### Modèle d'état (vs Recoil)

La différence entre Zustand et Recoil
est similaire à celle entre Zustand et Jotai.
Recoil dépend des clés de chaîne d'atomes
au lieu des identités référentielles d'objets d'atomes.
De plus, Recoil doit envelopper votre application dans un fournisseur de contexte.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  setCount: (countCallback: (count: State['count']) => State['count']) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  setCount: (countCallback) =>
    set((state) => ({ count: countCallback(state.count) })),
}))
```

**Recoil**

```ts
import { atom } from 'recoil'

const count = atom({
  key: 'count',
  default: 0,
})
```

### Optimisation du rendu (vs Recoil)

Similaire aux comparaisons d'optimisation précédentes,
Recoil effectue des optimisations de rendu grâce à la dépendance des atomes.
Alors qu'avec Zustand, il est recommandé que
vous appliquiez manuellement des optimisations de rendu en utilisant des sélecteurs.

**Zustand**

```ts
import { create } from 'zustand'

type State = {
  count: number
}

type Actions = {
  setCount: (countCallback: (count: State['count']) => State['count']) => void
}

const useCountStore = create<State & Actions>((set) => ({
  count: 0,
  setCount: (countCallback) =>
    set((state) => ({ count: countCallback(state.count) })),
}))

const Component = () => {
  const count = useCountStore((state) => state.count)
  const setCount = useCountStore((state) => state.setCount)
  // ...
}
```

**Recoil**

```ts
import { atom, useRecoilState } from 'recoil'

const countAtom = atom({
  key: 'count',
  default: 0,
})

const Component = () => {
  const [count, setCount] = useRecoilState(countAtom)
  // ...
}
```

## Tendance des téléchargements NPM

- [Tendance des téléchargements NPM des bibliothèques de gestion d'état pour React](https://npm-stat.com/charts.html?package=zustand&package=jotai&package=valtio&package=%40reduxjs%2Ftoolkit&package=recoil)