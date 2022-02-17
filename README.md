# Пример огранизации библиотеки

## Настраиваем разрешения в github

- Запрещаем пуш в main. Settings -> Branches

## Добавление semantic-release в проект на github.

[Ссылка на библиотеку](https://github.com/semantic-release/semantic-release)

Для чего:
- Автоматическое создание тегов с корректной версией ПО
- Коммиты, удовлетворяющие определенному формату

### Устанавливаем semantic-release

- npm install -g semantic-release-cli
- semantic-release-cli setup

### Убираем деплой на npm

- добавляем файл .releaserc.json
```json
{
  "plugins": [
    ["@semantic-release/npm", {
      "npmPublish": false
    }]
  ]
}
```

### Настраиваем ci через github actions

- Добавляем файл .github/workflows/release.yml

```yml
name: Release package
on:
  push:
    branches:
      - main

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: "16.x"
      - run: npm ci
      - run: npm run semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

В результате при каждом пуше в ветку main будет создаваться тег в репозитории github с соответстующей версией приложения.

Для корректной работы библиотеки все коммиты должны быть оформлены в строгом соответствии с [этим документом](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-format).

### Добавляем генерацию корректных коммитов

- Устанавливаем необходимые программы

```bash
npm install --save-dev commitizen
npm i cz-conventional-changelog --save-dev
```

- Модифицируем package.json

```json
{
  "scripts": {
    "commit": "cz",
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

Таким образом при запуске npm run commit будет генерироваться корректный коммит

### Добавляем линтер для коммитов

- Устанавливаем нужные пакеты

```bash
npm install --save-dev @commitlint/{cli,config-conventional}
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js

npm install husky --save-dev
npx husky install
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```

- Еще раз правим package.json

```json
"scripts": {
  "prepare": "husky install"
},
```
