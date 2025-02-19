---
title: 'App Configuration'
excerpt: 'Learn how to configure a Jovo app and how to add plugins, components, and staging.'
---

# App Configuration

The app configuration in `app.ts` is the place where you can add plugins, components, and other configurations to your Jovo app. [For project related configuration, take a look here](./project-config.md).

## Introduction

The app configuration files are the main entry point of your Jovo apps. Each Jovo project usually comes with at least two files for this:

- `app.ts`: Default configurations
- `app.dev.ts`: Configurations for local development ([FileDb](https://www.jovo.tech/marketplace/db-filedb), [Express server](https://www.jovo.tech/marketplace/server-express) and the [Jovo Debugger](https://www.jovo.tech/docs/debugger))

Jovo offers different [ways to add configurations](#ways-to-add-configurations), [many configuration options](#configuration-elements), and [staging](#staging) that makes it possible to have different Jovo app versions for different deployment environments.

## Ways to add Configurations

There are three ways how app configurations can be done:

- Using the `new App()` constructor in `app.ts` for default configurations.
- Using `app.configure()` for stage-specific configurations.
- Using `app.use()` to add specific plugins and components anywhere in the app.

In the `app.ts`, the configuration is added like this:

```typescript
import { App } from '@jovotech/framework';

// ...

const app = new App({
  // Configuration
});
```

On top of the default configuration, you can add [stages](#staging) with specific options that can be added like this, for example in an `app.dev.ts` file:

```typescript
import { app } from './app';
// ...

app.configure({
  // Configuration
});
```

Both the constructor and `configure()` support the full range of [configuration elements](#configuration-elements).

The third option is the `use()` method. It allows you to add plugins and components anywhere in the app:

```typescript
import { app } from './app';
import { SomePlugin } from './plugin';
// ...

app.use(
  new SomePlugin({
    // Configuration
  }),
);
```

## Configuration Elements

The configuration object that can be passed to both the constructor and the `configure()` method contains [components](#components), [plugins](#plugins), [logging](#logging), and [routing](#routing).

```typescript
{
  components: [
    // ...
  ],
  plugins: [
    // ...
  ],
  logging: {
    // ...
  },
  routing: {
    // ...
  }
}
```

### Components

You can [register root components](./components.md#register-root-components) with your app by adding them to the `components` array:

```typescript
import { GlobalComponent } from './components/GlobalComponent';

// ...

{
  components: [
    GlobalComponent
    // ...
  ],
  // ...
}
```

[Learn more about component registration here](./components.md#register-root-components).

### Plugins

You can add plugins to the `plugins` array like this:

```typescript
import { AlexaPlatform } from '@jovotech/platform-alexa';

// ...

{
  plugins: [
    new AlexaPlatform({
      // Configuration
    })
    // ...
  ],
  // ...
}
```

Each plugin has its own configuration options which you can find in the respective plugin's documentation.

Additionally, each plugin config includes a `skipTests` option that makes sure that [unit tests](https://www.jovo.tech/docs/unit-testing) don't use that plugin:

```typescript
{
  plugins: [
    new SomePlugin({
      // ...
      skipTests: true,
    })
  ],
}
```

You can also access a specific plugin like this:

```typescript
app.plugins.<PluginConstructor>

// Example
app.plugins.SomePlugin
```

This can be helpful if you want to add additional configurations to the default plugin config outside `app.ts`. See [staging](#staging) for more information.

### Logging

[Logging](./logging.md) is enabled by adding the following to the app config:

```typescript
{
  // ...

  logging: true,
}
```

You can also add granular configurations by turning `logging` into an object:

```typescript
{
  // ...

  logging: {
    // ...
  }
}
```

[Learn more about logging and its configuration options here](./logging.md).

### Routing

[Routing](./routing.md) configurations are added to the `routing` object:

```typescript
{
  routing: {
    intentMap: {},
    intentsToSkipUnhandled: [],
  },
  // ...
}
```

#### intentMap

Especially with apps that work across different platforms, it might happen that different platforms use different intent names.

`intentMap` provides a way to map incoming intents to a unified intent that can be used in your [handler routing](./handlers.md#handler-routing-and-the-handle-decorator).

```typescript
{
  routing: {
    intentMap: {
      'AMAZON.HelpIntent': 'HelpIntent',
      // ...
    },
    // ...
  },
  // ...
}
```

For platforms like Alexa that already come with an intent in their requests, the mapped intent name is added to the root of the [`$input` object](./input.md):

```typescript
{
  type: 'INTENT',
  intent: 'HelpIntent',
}
```

If you're using an [NLU integration](./nlu.md), the original intent stays in the `nlu` property and the mapped intent is added to the root of `$input`:

```typescript
{
  type: 'TEXT',
  text: 'My name is Max',
  nlu: {
    intent: 'MyNameIsIntent',
    entities: {
      name: {
        value: 'Max',
      },
    },
  },
  intent: 'MappedMyNameIsIntent',
}
```

#### intentsToSkipUnhandled

`intentsToSkipUnhandled` includes all intents that shouldn't be fulfilled by an [`UNHANDLED` handler](./handlers.md#unhandled).

```typescript
{
  routing: {
    intentsToSkipUnhandled: [
      'HelpIntent'
    ],
    // ...
  },
  // ...
}
```

[Learn more about `intentsToSkipUnhandled` here](./routing.md#intentstoskipunhadled).

## Staging

Stage-specific configurations from a file called `app.<stage>.ts` get merged into the default configuration from `app.ts`.

For example, most Jovo projects include an `app.dev.ts` file that comes with specific configuration for local development ([FileDb](https://www.jovo.tech/marketplace/db-filedb), [Express server](https://www.jovo.tech/marketplace/server-express) and the [Jovo Debugger](https://www.jovo.tech/docs/debugger)).

You can create a new stage like this:

```sh
$ jovo new:stage <stage>

# Example that creates a new app.prod.ts file
$ jovo new:stage prod
```

This creates a new file `app.prod.ts`. In the process, you can select plugins and a server integration to work with this stage.

Typically, a stage app config uses the `configure()` method to modify the configuration.

```typescript
import { app } from './app';
// ...

app.configure({
  // Configuration
});
```

It is also possible to reference a plugin from the default configuration in `app.ts` and add plugins to it using the `use()` method.

Here is an example for [Dashbot Analytics](https://www.jovo.tech/marketplace/analytics-dashbot) being added to [Alexa](https://www.jovo.tech/marketplace/platform) in `app.prod.ts`:

```typescript
import { app } from './app';
import { DashbotAnalytics } from '@jovotech/analytics-dashbot';
// ...

app.plugins.AlexaPlatform.use(new DashbotAnalytics({ apiKey: '<yourApiKey>' }));
```
