# NPM - NodeJs et PNPM - Linux

Il m'est souvent utile d'installer nodejs sur linux, pour cela il existe plusieurs solutions

## Utiliser pnpm

Le plus simple est sans doute d'utiliser directement pnpm. Il suffit de l'installer via

```
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

On peut ensuite l'utiliser pour choisir la version de nodejs que l'on veut, par exemple:

```
pnpm env add --global lts
pnpm env use --global lts
```

On peut à la place de lts mettre la version 16 ou 18 par exemple.

