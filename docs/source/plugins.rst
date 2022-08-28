Plugins
=======

Plugins are used to extend the functionality of the FRC Web Components dashboard. The main components that can be extended are:

**Elements**
   These are UI components like buttons, line charts and robot visualizations that are used to build dashboards. Users have the option of adding their own custom elements in addition to the ones that ship with the dashboard. Read more about elements :ref:`here <Elements>`.
**Source Providers**
   Source providers are what provide data to your elements, such as NetworkTables and Camera Streams. You can create your own Source providers to provide additional sources of data to your elements. Read more about Source providers :ref:`here <Source Providers>`.


Elements
--------

Stuff about components

Source Providers
----------------

Source Providers are used to provide data to your elements. For example the NetworkTables source provider can be used to control dashboard elements through NetworkTables:

.. image:: ./assets/plugins-source-provider1.*

Above the FRC Accelerometer element's source provider is "NetworkTables" with the key "/accel". This will map entries in the "/accel" subtable to the accelerometer element's properties.

Let's create a source provider which can be used to control a number slider:

.. image:: ./assets/plugins-source-provider2.*

First, create a file called `slider-provider.ts` in the providers folder:

.. image:: ./assets/plugins-source-provider3.*

Paste the following code into `slider-provider.ts`:

.. code-block:: typescript

   import { SourceProvider } from '@webbitjs/store';

   export default class SliderProvider extends SourceProvider {
      constructor() {
         super();
         this.updateSource('/slider/value', 3);
         this.updateSource('/slider/min', 0);
         this.updateSource('/slider/max', 10);
      }
   }


Now add the following lines to `index.ts`:

.. code-block:: typescript
   
   // This is an import that goes at the top of the file
   import SliderProvider from './providers/slider-provider';

.. code-block:: typescript
   
   // This is how the plugin adds the source provider to the dashboard
   dashboard.addSourceProvider('SliderProvider', new SliderProvider());


Your `index.ts` file should now look something like this:

.. image:: ./assets/plugins-source-provider4.*

You can now use the provider on your dashboard's elements:

.. image:: ./assets/plugins-source-provider5.*

Amazing! This would be a lot more interesting if the data came from an external source, however. Let's update SliderProvider so that the sources can be updated in the browser's URL:

.. code-block:: typescript

   import { SourceProvider } from '@webbitjs/store';

   function getHash(): Record<string, string> {
      const keyValuePairs: Record<string, string> = {};
      const hash = new URL(document.URL).hash.substring(1);
      hash.split('&').forEach(keyValue => {
         const [key, value] = keyValue.split('=');
         keyValuePairs[key] = value;
      });
      return keyValuePairs;
   }

   export default class SliderProvider extends SourceProvider {
      constructor() {
         super();
         this.updateSliderValues();
         window.addEventListener('hashchange', () => { 
            this.updateSliderValues();
         });
      }

      private updateSliderValues() {
         const hash = getHash();
         ['value', 'min', 'max'].forEach(key => {
            const value = parseFloat(hash[key]);
            if (!isNaN(value)) {
            this.updateSource(`/slider/${key}`, value);
            }
         });
      }
   }

Now the slider can be controlled through the browser's URL:

.. image:: ./assets/plugins-source-provider6.*

It would be great if moving the slider also updated the URL as well. Let's update SliderProvider to make it do that by adding the following code to `slider-provider.ts`:

.. code-block:: typescript

   function updateHash(key: string, value: string) {
      const newHash = {
         ...getHash(),
         [key]: value
      };
      const hashString = Object.entries(newHash)
         .map(([key, value]) => `${key}=${value}`)
         .join('&');
      document.location.hash = `#${hashString}`;
   }


Next we will also be overriding the source provider's `userUpdate` method. The `userUpdate` method is called automatically when a change to an element's properties is detected. The `key` is the source key currently bound to the element's property. The `userUpdate` method has a default implementation which simply calls `updateSource` with the key value pair passed into `userUpdate`:

.. code-block:: typescript

   userUpdate(key: string, value: unknown) {
      this.updateSource(key, value);
   }


Instead of setting the source directly, we will instead be updating the URL hash string by calling the `updateHash` function:

.. code-block:: typescript

   userUpdate(key: string, value: unknown) {
      const numberValue = parseFloat(value as any);
      const property = ['value', 'min', 'max'].find(prop => key.endsWith(prop));
      if (!isNaN(numberValue) && property) {
         updateHash(property, numberValue.toString());
      }
  }

The `slider-provider.ts` file should now have the following code:

.. code-block:: typescript

   import { SourceProvider } from '@webbitjs/store';

   function getHash(): Record<string, string> {
      const keyValuePairs: Record<string, string> = {};
      const hash = new URL(document.URL).hash.substring(1);
      hash.split('&').forEach(keyValue => {
         const [key, value] = keyValue.split('=');
         keyValuePairs[key] = value;
      });
      return keyValuePairs;
   }

   function updateHash(key: string, value: string) {
      const newHash = {
         ...getHash(),
         [key]: value
      };
      const hashString = Object.entries(newHash)
         .map(([key, value]) => `${key}=${value}`)
         .join('&');
      document.location.hash = `#${hashString}`;
   }

   export default class SliderProvider extends SourceProvider {
      constructor() {
         super();
         this.updateSliderValues();
         window.addEventListener('hashchange', () => {
            this.updateSliderValues();
         });
      }

      userUpdate(key: string, value: unknown) {
         const numberValue = parseFloat(value as any);
         const property = ['value', 'min', 'max'].find(prop => key.endsWith(prop));
         if (!isNaN(numberValue) && property) {
            updateHash(property, numberValue.toString());
         }
      }

      private updateSliderValues() {
         const hash = getHash();
         ['value', 'min', 'max'].forEach(key => {
            const value = parseFloat(hash[key]);
            if (!isNaN(value)) {
               this.updateSource(`/slider/${key}`, value);
            }
         });
      }
   }

The URL should now be updated when you move the slider:

.. image:: ./assets/plugins-source-provider7.*
