---
title: Introduction
description: Comment utiliser Zustand
nav: 0
---

<div class="flex justify-center mb-4">
  <img src="../../bear.jpg" alt="Logo Zustand" />
</div>

Une solution de gestion d'état minimaliste, rapide et évolutive.
Zustand possède une API confortable basée sur les hooks.
Elle n'est pas verbeuse ni dogmatique,
mais a suffisamment de conventions pour être explicite et similaire à Flux.

Ne la sous-estimez pas parce qu'elle est mignonne, elle a des griffes !
Beaucoup de temps a été consacré à résoudre les problèmes courants,
comme le redouté [problème des enfants zombies],
la [concurrence de React], et la [perte de contexte]
entre les différents moteurs de rendu.
C'est peut-être le seul gestionnaire d'état dans l'écosystème React qui résout correctement tous ces problèmes.

Vous pouvez essayer une démo en direct [ici](https://codesandbox.io/s/dazzling-moon-itop4).

[problème des enfants zombies]: https://react-redux.js.org/api/hooks#stale-props-and-zombie-children
[concurrence de React]: https://github.com/bvaughn/rfcs/blob/useMutableSource/text/0000-use-mutable-source.md
[perte de contexte]: https://github.com/facebook/react/issues/13332

## Installation

Zustand est disponible en tant que package sur NPM :

```bash
# NPM
npm install zustand
# Ou utilisez n'importe quel gestionnaire de paquets de votre choix.
```

## Commencez par créer un store

Votre store est un hook !
Vous pouvez y mettre n'importe quoi : primitives, objets, fonctions.
La fonction `set` _fusionne_ l'état.

```js
import { create } from 'zustand'

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
  updateBears: (newBears) => set({ bears: newBears }),
}))
```

## Puis liez vos composants, et c'est tout !

Vous pouvez utiliser le hook n'importe où, sans avoir besoin de providers.
Sélectionnez votre état et le composant consommateur
se re-rendra lorsque cet état change.

```jsx
function BearCounter() {
  const bears = useStore((state) => state.bears)
  return <h1>{bears} ours dans les environs...</h1>
}

function Controls() {
  const increasePopulation = useStore((state) => state.increasePopulation)
  return <button onClick={increasePopulation}>ajouter un</button>
}
```