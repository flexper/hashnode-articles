---
title: "Rosetty React: The Complete Internationalization Solution for React Applications"
datePublished: Mon Mar 20 2023 14:00:38 GMT+0000 (Coordinated Universal Time)
cuid: clfgw9oef01mgrrnv9x0k91ng
slug: rosetty-react-the-complete-internationalization-solution-for-react-applications
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678891338212/c85fe207-98a5-4f7a-96f4-06bfa8d58999.png
tags: reactjs, i18n, translation, nextjs

---

If you're building a React application that needs to support multiple languages, then you know that internationalization (i18n) can be a complex and time-consuming task. Fortunately, there are libraries like Rosetty React that can help simplify the process and make it much easier to manage translations and language-specific content.

In this article, we'll take a closer look at Rosetty React and how you can use it to internationalize your React applications.

## What is Rosetty React?

Rosetty React is a complete internationalization (i18n) solution for React applications that's built on top of the Rosetty library. It provides a set of React components and hooks that make it easy to manage translations and language-specific content in your application.

With Rosetty React, you can define a set of dictionaries for each language that your application supports, and then use a set of React components and hooks to access and render the appropriate translations and content based on the user's selected language.

## Why use Rosetty React?

There are many benefits to using Rosetty React for internationalizing your React applications. Some of the key benefits include:

1. Simplified Translation Management
    

With Rosetty React, you can define a set of dictionaries for each language that your application supports. These dictionaries contain all the translations for your application, and you can easily manage them using a simple JSON file.

1. Dynamic Language Switching
    

Rosetty React makes it easy to support multiple languages in your application, and it provides a set of hooks and components that allow you to dynamically switch between languages based on user preferences.

1. Lightweight and Fast
    

Rosetty React is built with performance in mind, and it's designed to be lightweight and fast. It has a small footprint and minimal dependencies, which means that it won't slow down your application or impact performance.

1. Easy to Use
    

Rosetty React is designed to be easy to use, even for developers who are new to internationalization. It provides a simple and intuitive API that makes it easy to manage translations and language-specific content in your application.

## How to use Rosetty React

Now that we've covered some of the key benefits of Rosetty React, let's take a closer look at how you can use it in your React applications.

**Step 1: Install Rosetty React**

The first step is to install the Rosetty React package using npm or yarn:

```bash
npm install rosetty rosetty-react
```

**Step 2: Define Your Language Dictionaries**

The next step is to define a set of dictionaries for each language that your application supports. These dictionaries should contain all the translations for your application, and they should be stored in a simple JSON format.

Here's an example of what a dictionary might look like for the French language:

```json
{
  "hello": "Bonjour",
  "welcome": "Bienvenue",
  "goodbye": "Au revoir"
}
```

You can define as many dictionaries as you need, one for each language that your application supports.

**Step 3: Create the RosettyProvider**

Once you've defined your dictionaries, you can create a `RosettyProvider` component that will provide the translations and language-specific content to your application.

```typescript
import React from 'react';
import { RosettyProvider, locales as rosettyLocales } from 'rosetty-react';

const locales = { fr: { dict: { home : 'home' }, locale: rosettyLocales.fr } };
const defaultLanguage = 'fr';

const App = ({ children }) => (
  <RosettyProvider locales={locales} defaultLanguage={defaultLanguage}>
    {children}
  </RosettyProvider>
);

export default App;
```

## Using `useRosetty`

The `useRosetty` hook is the most commonly used method for accessing localized content within your components. Here's an example of how to use it:

```typescript
import { useRosetty } from 'rosetty-react';
import frDict from '../i18n/fr';

export const useI18n = () => {
  return useRosetty<typeof frDict>(); //Enable autocompletion base on you translation file
};
const Home = () => {
  const { t } = useI18n();

  return <h1>{t('home')}</h1>;
};

export default Home;
```

In the example above, `useRosetty` is imported from the `rosetty-react` package. This hook provides a `t` function that can be used to translate strings based on the current locale.

## Changing the Language

You may want to allow the user to switch the application's language dynamically. To achieve this, you can use the `setLanguage` function provided by the `RosettyContext`.

Here's an example:

```typescript
import { RosettyContext } from 'rosetty-react';

export const useI18n = () => {
  return useRosetty<typeof frDict>(); //Enable autocompletion base on you translation file
};

const LanguageSwitcher = () => {
  const { changeLang, actualLang } = useI18n();

  const handleChange = (event) => {
    changeLang(event.target.value);
  };

  return (
    <div>
      <label htmlFor="language-switcher">Choose a language:</label>
      <select
        id="language-switcher"
        value={actualLang}
        onChange={handleChange}
      >
        <option value="en">English</option>
        <option value="fr">Français</option>
      </select>
    </div>
  );
};

export default LanguageSwitcher;
```

In the example above, we import `RosettyContext` from `rosetty-react` and use the `useContext` hook to access the context object. We then extract the `changeLang` function and the `actualLang` state value from the context.

The `handleChange` function is called when the user selects a new language from the dropdown menu. This function updates the language by calling `changeLan` with the new language value.

## Conclusion

Rosetty-React is an excellent localization library that makes it easy to add multi-language support to your React application. Its simple API and lightweight design make it easy to integrate into your existing project without weighing it down.

Whether you're building a simple website or a complex application, Rosetty-React is a great choice for managing localized content in React. Its straightforward approach to internationalization makes it easy to get started, while its powerful features ensure that your application can support any language or locale with ease.