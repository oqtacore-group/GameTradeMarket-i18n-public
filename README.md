# [GameTradeMarket PROD](https://gametrade.market) / [GameTradeMarket QA](https://qa.gametrade.market)

The directory structure (`locales/*alpha2*`) must be preserved, as when building GameTradeMarket Frontend (executing the `./cli.sh downloadLocales` function), it accesses the repository and extracts the folder structure into the project directory `packages/app/public/locales`

File creation must be duplicated in each localization directory (`locales/en` and `locales/*otherLang*`)

Each locale must have a (even empty) `common` file - which is required for `i18n`

File placement: `locales/{code-alpha2}/*`

Example file name `{page|(component/element/modifier)}.json`:

```
  "language": "{language}",
  "i18n": "{code-alpha2}",
  "ns": "base",
  "translation": {},
```

- `language` - language name in English with **lowercase** first letter
- `i18n` - language code in **Alpha2** format ([you can check here](https://www.artlebedev.ru/country-list/))
- `ns` - default field
- `translation` - object where translations for the language are stored

## New file for a page

After adding a new file for a page, you need to update the frontend configuration in the page's connection file

```tsx
import { serverSideTranslations } from "next-i18next/serverSideTranslations";
import nextI18NextConfig from "@game-trade/config/next/i18next.js";

export async function getServerSideProps(ctx: any) {
  const { data } = await api.query<LoopbackQuery, LoopbackQueryVariables>({
    query: LoopbackDocument,
    fetchPolicy: "no-cache",
  });

  return {
    props: {
      ...(await serverSideTranslations(
        context.locale ?? context.defaultLocale,
        // Connecting files for the page
        ["{nameFile}Page", ..."{otherNameFiles}Page"],
        // Connecting files for the page
        nextI18NextConfig
      )),
      serverSideData: data,
    },
  };
}
```

Connection on the page itself:

```tsx
import { useTranslation } from "next-i18next";

export const DevelopersContainer = () => {
  // nameFile - name of the connection file for the page
  const { t } = useTranslation("{newNameFile}Page", {
    keyPrefix: "translation",
  });

  return <p>{t("title")}</p>;
};
```

## New file for a component / function

```ts
i18next
  .use(backend)
  .use(initReactI18next)
  .init({
    fallbackLng: "en",
    debug: false,
    //   Connecting component / function
    ns: [
      "modifier",
      "elements",
      "common",
      "buttons",
      "{newNameFile}(Element|Component)",
    ],
    //   Connecting component / function
    defaultNS: false,
    react: {
      useSuspense: false,
    },
    interpolation: {
      escapeValue: false,
    },
  });
```

## GameTradeMarket Frontend download function `./cli.sh`

You need to add the variable `GITHUB_LANGUAGES_PERSONAL_ACCESS_TOKEN_CLASSIC` to the `packages/config/.env.*` files with the value obtained from the [github interface](https://github.com/settings/tokens)

```sh
downloadLocales() {
  echo "START downloadLocales"
  branch=$(git rev-parse --symbolic-full-name --abbrev-ref HEAD)
#  branch="HEAD"
  if [[ $branch == "release" ]]
    then
      echo 'download locales env.prod'
      source ./packages/config/.env.prod
  elif [[ $branch == 'HEAD' ]]
    then
      echo 'download locales server'
  elif [[ $branch ]]
    then
      echo 'download locales env.dev'
      source ./packages/config/.env.dev
  fi

  echo 'github language token variable:' + $GITHUB_LANGUAGES_PERSONAL_ACCESS_TOKEN_CLASSIC

  rm -r ./packages/app/public/locales/*
  wait
  curl -H 'Authorization: token '$GITHUB_LANGUAGES_PERSONAL_ACCESS_TOKEN_CLASSIC \
       -H 'Accept: application/vnd.github.v3.raw' \
       -O \
       -L https://github.com/oqtacore-group/GameTradeMarket-languages/archive/main.zip -o main.zip && unzip main.zip "GameTradeMarket-languages-main/locales/*" -d ./packages/app/public/locales && rm main.zip
  wait
  mv -f ./packages/app/public/locales/GameTradeMarket-languages-main/locales/* ./packages/app/public/locales/
  wait
  rm -r ./packages/app/public/locales/GameTradeMarket-languages-main/
#  curl -L https://github.com/oqtacore-group/GameTradeMarket-languages/archive/main.zip -o main.zip && unzip main.zip "GameTradeMarket-languages-main/locales/*" -d ./packages/app/public/locales && mv ./packages/app/public/locales/GameTradeMarket-languages-main/locales/* ./packages/app/public/locales/ && rm main.zip
#  node_modules/.bin/wget https://raw.githubusercontent.com/oqtacore-group/GameTradeMarket-languages/main/common.en.json -d packages/app/public/locales/en/common.json
  wait
  echo "All done downloadLocales"
}
```

# License

This project is licensed under the [Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/).

This means you are free to:

- Share — copy and redistribute the material in any medium or format
- Adapt — remix, transform, and build upon the material

Under the following terms:

- Attribution — You must give appropriate credit, provide a link to the license, and indicate if changes were made.
- NonCommercial — You may not use the material for commercial purposes.
