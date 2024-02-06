# API Documentation

> [!WARNING]
> This documentation is made for those using the experimental new Frontend system.
> If you are not using the new frontend system, please go [here](./README.md).

This is an extension for the catalog plugin that provides components to discover and display API entities.
APIs define the interface between components, see the [system model](https://backstage.io/docs/features/software-catalog/system-model) for details.
They are defined in machine readable formats and provide a human readable documentation.

The plugin provides a standalone list of APIs, as well as an integration into the API tab of a catalog entity.

![Standalone API list](./docs/api_list.png)
![OpenAPI Definition](./docs/openapi_definition.png)
![Integration into components](./docs/entity_tab_api.png)

Right now, the following API formats are supported:

- [OpenAPI](https://swagger.io/specification/) 2 & 3
- [AsyncAPI](https://www.asyncapi.com/docs/reference/specification/latest)
- [GraphQL](https://graphql.org/learn/schema/)

Other formats are displayed as plain text, but this can easily be extended.

To fill the catalog with APIs, [provide entities of kind API](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-api).
To link that a component provides or consumes an API, see the [`providesApis`](https://backstage.io/docs/features/software-catalog/descriptor-format#specprovidesapis-optional) and [`consumesApis`](https://backstage.io/docs/features/software-catalog/descriptor-format#specconsumesapis-optional) properties on the Component kind.

## Installation

> The plugin is already added when using `npx @backstage/create-app` so you can skip these steps.

1. Install the API docs plugin

```bash
# From your Backstage root directory
yarn --cwd packages/app add @backstage/plugin-api-docs
```

2. In your application's configuration file, enable the api docs entity cards and tabs so that the they begins to be presented on the catalog entity page:

```yaml
# app-config.yaml
app:
  experimental:
    # Auto discovering all plugins extensions
    packages: all
  extensions:
    # Enabling some entity Cards
    # The cards will be displayed in the same order it appears in this setting list
    # Shows a table of components that provides a particular api
    - entity-card:api-docs/providing-components:
        config:
          # Presenting the card ony for entites of kind api
          filter: kind:api
    # Shows a table of components that consumes a particular api
    - entity-card:api-docs/consuming-components:
        config:
          # Presenting the card ony for entites of kind api
          filter: kind:api
    # Enabling some Contents
    # The contents will be displayed in the same order it appears in this setting list
    # Shows a "Definition" tab for entities of kind api
    - entity-content:api-docs/definition
    # Shows an "Apis" tab for entities of kind component
    - entity-content:api-docs/apis
```

3. Then start the app, navigate to an entity's page and see the Relations graph there;
4. You can also access the Apis explorer page by clicking on the "APIs" sidebar item.

## Customization

### Packages

The features of this plug-in can be discovered automatically as soon as you install it, but you can also configure the plug-in not to be enabled for certain [environments](https://backstage.io/docs/conf/writing/#configuration-files). See the examples below:

_Enabling auto discovering the plugin extensions in production_

```yaml
# app-config.production.yaml
# Overriding configurations for the local production environment
app:
  experimental:
    packages:
      # Only the following packages will be included
      include:
        - '@backstage/plugin-api-docs'
```

_Disabling auto discovering the plugin extensions in development_

```yaml
# app-config.local.yaml
# Overriding configurations for the local development environment
app:
  experimental:
    packages:
      # All but the following package will be included
      exclude:
        - '@backstage/plugin-api-docs'
```

For more options of package configurations, see [this](https://backstage.io/docs/frontend-system/architecture/app/#feature-discovery) documentation.

### Routes

The Catalog graph plugin exposes regular and external routes that can be used to configure route bindings.

| Key           | Type           | Description                            |
| ------------- | -------------- | -------------------------------------- |
| `root`        | Regular route  | A route ref to the Apis Explorer page. |
| `registerApi` | External route | A route ref to Register Api page.      |

As an example, here is an association between the external register api page and a regular route from other plugin:

```yaml
# app-config.yaml
app:
  routes:
    bindings:
      # example binding the external register api route
      catalog-graph.registerApi: <plugin-id>.<regular-route-key>
```

Additionally, it is possible to point a route from another plugin to the Apis explorer page:

```yaml
# app-config.yaml
app:
  routes:
    bindings:
      # example binding a external route to the apis explorer page
      <plugin-id>.<external-route-key>: api-docs.root
```

Route binding is also possible through code. For more information, see [this](https://backstage.io/docs/frontend-system/architecture/routes#binding-external-route-references) documentation.

### Extensions

#### Nav Item

A sidebar navigation item that links to Apis Explorer page.

##### Disable

Hide the "APIs" sidebar item by adding the following configuration:

```yaml
# app-config.yaml
app:
  extensions:
    # this is the extension id and it follows the naming pattern bellow:
    # <extension-kind>/<plugin-namespace>:<extension-name>
    # example disbaling the apis docs nav item extension
    - nav-item:api-docs: false
```

##### Config

Define configurations for the nav item under the `app.extensions.nav-item:api-docs.config` key in the `app-config.yaml` file. Here are the configurations available:

```yaml
# app-config.yaml
app:
  extensions:
    # this is the extension id and it follows the naming pattern bellow:
    # <extension-kind>/<plugin-namespace>:<extension-name>
    # example configuring the apis docs nav item extension
    - nav-item:api-docs:
        config:
          # The nav item title text, defaults to "APIs"
          title: 'Apis Explorer'
          # The nav item path text, defaults to "/api-docs"
          path: '/apis-explorer
```

#### Override

Only the icon is not customisable via `app-config.yaml` file, so in ordem to changing it, you have to override the Api docs nav item extension:

Here is an example overriding the nav item extension with a custom component:

```tsx
import { createExtensionOverrides, createNavItem, createSchemaFromZod } from '@backstage/backstage-plugin-api';

export default createExtensionOverrides(
  extensions: [
    createNavItemExtension({
      // These namespace is necessary so the system knows that this extension will override the default nav item provided by the 'api-docs' plugin
      namespace: 'api-docs',
      // It's your choice whether to use the original extension's title or a different one
      title: 'APIs',
      // To continue pointing to the same page as the original extension, use the same route ref
      routeRef: convertLegacyRouteRef(rootRoute),
      // Custom icon components are loaded here
      loader: () => import('./components').then(m => <m.MyCustomApiDocsComponent />)
    })
  ]
);
```

For more information about where to place extension overrides, see the official [documentation](https://backstage.io/docs/frontend-system/architecture/extension-overrides).

#### Config Api

This is an api used by tha api docs plugin to get the api definition widget. To personalize the widgets returned by this api, you should use extension overrides.
For more information about where to place extension overrides, see the official [documentation](https://backstage.io/docs/frontend-system/architecture/extension-overrides).

##### Custom Api Renderings

You can add support for additional API types by providing a custom implementation for the `apiDocsConfigRef`.
You can also use this to override the rendering of one of the already supported types.

This is an example with a made-up renderer for SQL schemas:

```tsx
import {
  createExtensionOverrides,
  createApiExtenion,
  createApiFactory,
} from '@backstage/frontend-plugin-api';
import { ApiEntity } from '@backstage/catalog-model';
import {
  apiDocsConfigRef,
  ApiDefinitionWidget,
  defaultDefinitionWidgets,
} from '@backstage/plugin-api-docs';
import { SqlRenderer } from '...';

export default createExtensionOverrides({
  extensions: [
    createApiExtenion({
      factory: createApiFactory({
        api: apiDocsConfigRef,
        deps: {},
        factory: () => {
          // load the default widgets
          const definitionWidgets = defaultDefinitionWidgets();
          return {
            getApiDefinitionWidget: (apiEntity: ApiEntity) => {
              // custom rendering for sql
              if (apiEntity.spec.type === 'sql') {
                return {
                  type: 'sql',
                  title: 'SQL',
                  component: definition => (
                    <SqlRenderer definition={definition} />
                  ),
                } as ApiDefinitionWidget;
              }

              // fallback to the defaults
              return definitionWidgets.find(
                d => d.type === apiEntity.spec.type,
              );
            },
          };
        },
      }),
    }),
  ],
});
```

##### Adding `requestInterceptor` to Swagger UI

To configure a [`requestInterceptor` for Swagger UI](https://github.com/swagger-api/swagger-ui/tree/master/flavors/swagger-ui-react#requestinterceptor-proptypesfunc) you'll need to add the following to your `api.tsx`:

```tsx
import {
  createExtensionOverrides,
  createApiExtenion,
  createApiFactory,
} from '@backstage/frontend-plugin-api';
import {
  apiDocsConfigRef,
  defaultDefinitionWidgets,
  OpenApiDefinitionWidget,
} from '@backstage/plugin-api-docs';
import { ApiEntity } from '@backstage/catalog-model';

export default createExtensionOverrides({
  extensions: [
    createApiExtenion({
      factory: createApiFactory({
        api: apiDocsConfigRef,
        deps: {},
        factory: () => {
          // Overriding openapi definition widget to add header
          const requestInterceptor = (req: any) => {
            req.headers.append('myheader', 'wombats');
            return req;
          };
          const definitionWidgets = defaultDefinitionWidgets().map(obj => {
            if (obj.type === 'openapi') {
              return {
                ...obj,
                component: definition => (
                  <OpenApiDefinitionWidget
                    definition={definition}
                    requestInterceptor={requestInterceptor}
                  />
                ),
              };
            }
            return obj;
          });

          return {
            getApiDefinitionWidget: (apiEntity: ApiEntity) => {
              return definitionWidgets.find(
                d => d.type === apiEntity.spec.type,
              );
            },
          };
        },
      }),
    }),
  ],
});
```

In the same way as the `requestInterceptor` you can override any property of Swagger UI.

##### Provide Specific Supported Methods to Swagger UI

This can be done through utilising the
[supportedSubmitMethods prop](https://github.com/swagger-api/swagger-ui/tree/master/flavors/swagger-ui-react#supportedsubmitmethods-proptypesarrayofproptypesoneofget-put-post-delete-options-head-patch-trace).
If you wish to limit the HTTP methods available for the `Try It Out` feature of an OpenAPI API
component, you will need to add the following to your `api.tsx`, listing the permitted methods for
your API in the `supportedSubmitMethods` parameter:

```tsx
import {
  createExtensionOverrides,
  createApiExtenion,
  createApiFactory,
} from '@backstage/frontend-plugin-api';
import {
  apiDocsConfigRef,
  defaultDefinitionWidgets,
  OpenApiDefinitionWidget,
} from '@backstage/plugin-api-docs';
import { ApiEntity } from '@backstage/catalog-model';

export default createExtensionOverrides({
  extensions: [
    createApiExtenion({
      factory: createApiFactory({
        api: apiDocsConfigRef,
        deps: {},
        factory: () => {
          const supportedSubmitMethods = ['get', 'post', 'put', 'delete'];
          const definitionWidgets = defaultDefinitionWidgets().map(obj => {
            if (obj.type === 'openapi') {
              return {
                ...obj,
                component: definition => (
                  <OpenApiDefinitionWidget
                    definition={definition}
                    supportedSubmitMethods={supportedSubmitMethods}
                  />
                ),
              };
            }
            return obj;
          });

          return {
            getApiDefinitionWidget: (apiEntity: ApiEntity) => {
              return definitionWidgets.find(
                d => d.type === apiEntity.spec.type,
              );
            },
          };
        },
      }),
    }),
  ],
});
```

N.B. if you wish to disable the `Try It Out` feature for your API, you can provide an empty list to
the `supportedSubmitMethods` parameter.

#### Explore Page

This plugin install an "Apis Explore" page extension that helps you to filter and visualize apis register in the Backstage software catalog.

| kind   | Namespace  | Name | Id              |
| ------ | ---------- | ---- | --------------- |
| `page` | `api-docs` | -    | `page:api-docs` |

##### Disable/Enable

This page is enable by default when you install the plugin, for disabling set the extension `disable` value to false:

> [!CAUTION]
> The `api-docs` plugin also install a sidebar item that points to this page, make sure you disable the item as well because otherwise the item will point to a not found page.

```yaml
# app-config.yaml
app:
  extensions:
    # this is the extension id and it follows the naming pattern bellow:
    # <extension-kind>/<plugin-namespace>:<extension-name>
    - page:api-docs: false
    # or
    # - page:api-docs:
    #     disable: true
```

To enable the extension again, simple remove the previous `page:api-docs: false` configuration or do:

```yaml
# app-config.yaml
app:
  extensions:
    # this is the extension id and it follows the naming pattern bellow:
    # <extension-kind>/<plugin-namespace>:<extension-name>
    - page:api-docs
    # or
    # - page:api-docs: true
    # or
    # - page:api-docs:
    #     disable: true
```

##### Override

This page extension implementation is overridable in situations when it's default implementation is not customizable enought.
Here is an example overriding the apis explorer page component:

````tsx
import { createExtensionOverrides, createPageExtension, createSchemaFromZod } from '@backstage/backstage-plugin-api';

export default createExtensionOverrides(
  extensions: [
    createPageExtension({
      // These namespace is necessary so the system knows that this extension will override the default explorer provided by the 'api-docs' plugin
      namespace: 'api-docs',
      // Ommitting name since we are overriding a plugin index page
      // It is up to you use the same original default path or not, but be aware that that links that are hardcoded to default path will stop working in case you choose to use a different path
      defaultPath: '/api-docs',
      // Using the original route ref because associating the page with a different route ref can cause the sidebar item or external plugin routes to point to a page that does not exist
      routeRef: convertLegacyRouteRef(rootRoute),
      // Custom page components are loaded here
      loader: () => import('./components').then(m => <m.MyCustomApiExplorerPage />)
    })
  ]
);

##### Has Apis Entity Card

An Entity Card Extension that render a table of components that are an api relation to a particular Software catalog Component.

###### Disable/Enable

This card is disabled by default when you install the `api-docs` plugin, but if you want make sure the card will always be disabled independently of the extension default definition, add this confoguration

```yaml
# app-config.yaml
app:
  extensions:
    # this is the extension id and it follows the naming pattern bellow:
    # <extension-kind>/<plugin-namespace>:<extension-name>
    # example disbaling the apis docs has apis entity card extension
    - nav-item:api-docs: false
````

##### Definition Entity Card

An Entity Card extension that renders an entity api definition.

##### Provided Apis Entity Card

An Entity Card extension that renders a table of apis provided by a particular Software Catalog Component.

##### Consumed Apis Entity Card

An Entity Card extension that renders a table of apis consumed by a particular Software Catalog Component.

##### Consuming Components Entity Card

An Entity Card extension that renders a table of components that consumes a particular Software Catalog api.

##### Providing Components Entity Card

An Entity Card extension that renders a table of components that provides a particular Software Catalog api.

##### Definition Entity Content

An Entity Content extension that renders a tab in the entity page showing a particular entity api definition.

##### Apis Entity Content

An Entity Content extension that renders a tab in the entity page showing a particular entity api definition.

### Integrations

#### Implementing OAuth 2 Authorization Code flow with Swagger UI

##### Adding `oauth2-redirect.html` to support OAuth2 `redirect_uri` route

The Swagger UI package by expects to have a route to `/oauth2-redirect.html` which processes
the redirect callback for the OAuth2 Authorization Code flow, however, this file is not installed
by this plugin.

Grab a copy of [`oauth2-redirect.html`](https://github.com/swagger-api/swagger-ui/blob/master/dist/oauth2-redirect.html)
and put it in the `app/public/` directory in order to enable Swagger UI to complete this redirection.

This also may require you to adjust `Content Security Policy` header settings of your Backstage application, so that the script in `oauth2-redirect.html` can be executed. Since the script is static we can add the hash of it directly to our CSP policy, which we do by adding the following to the `csp` section of the app configuration:

```yaml
script-src:
  - "'self'"
  - "'unsafe-eval'" # this is required for scaffolder usage, and ajv validation.
  - "'sha256-GeDavzSZ8O71Jggf/pQkKbt52dfZkrdNMQ3e+Ox+AkI='" # oauth2-redirect.html
```

##### Configuring your OAuth2 Client

You'll need to make sure your OAuth2 client has been registered in your OAuth2 Authentication Server (AS)
with the appropriate `redirect_uris`, `scopes` and `grant_types`. For example, if your AS supports
the [OAuth 2.0 Dynamic Client Registration Protocol](https://tools.ietf.org/html/rfc7591), an example
POST request body would look like this:

```json
{
    "client_name": "Example Backstage api-docs plugin Swagger UI Client",
    "redirect_uris": [
        "https://www.getpostman.com/oauth2/callback",
        "http://localhost:3000/oauth2-redirect.html"
        "https://<yourhost>/oauth2-redirect.html"
    ],
    "scope": "read_pets write_pets",
    "grant_types": [
        "authorization_code"
    ]
}
```

The above `redirect_uris` are:

- [Postman](https://www.postman.com/) testing: `https://www.getpostman.com/oauth2/callback`
- Local Backstage app development: `http://localhost:3000/oauth2-redirect.html`
- Backstage app production: `https://<yourhost>/oauth2-redirect.html`

##### Configuring OAuth2 in your OpenAPI 3.0 schema

To configure [OAuth 2 Authorization Code](https://swagger.io/docs/specification/authentication/oauth2/) flow
in your OpenAPI 3.0 schema you'll need something like this snippet:

```yaml
components:
  securitySchemes:
    oauth:
      type: oauth2
      description: OAuth2 service
      flows:
        authorizationCode:
          authorizationUrl: https://api.example.com/oauth2/authorize
          tokenUrl: https://api.example.com/oauth2/token
          scopes:
            read_pets: read your pets
            write_pets: modify pets in your account
security:
  oauth:
    - [read_pets, write_pets]
```
