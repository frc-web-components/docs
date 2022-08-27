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

First, create a file called `my-provider.ts` in the providers folder:

.. image:: ./assets/plugins-source-provider3.*

Paste the following code into `my-provider.ts`:

.. code-block:: typescript

   import { SourceProvider } from '@webbitjs/store';

   export default class MyProvider extends SourceProvider {
      constructor() {
         super();
         this.updateSource('/value', 5);
         this.updateSource('/min', 0);
         this.updateSource('/max', 10);
      }
   }

Now add the following lines to `index.ts`:

.. code-block:: typescript
   
   // This is an import that goes at the top of the file
   import MyProvider from './providers/my-provider';

.. code-block:: typescript
   
   // This is how the plugin adds the source provider to the dashboard
   dashboard.addSourceProvider('MyProvider', new MyProvider());


Your `index.ts` file should now look something like this:

.. image:: ./assets/plugins-source-provider4.*
