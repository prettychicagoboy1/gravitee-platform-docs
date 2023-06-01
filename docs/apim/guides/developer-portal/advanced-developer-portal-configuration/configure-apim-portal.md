# Configure APIM Portal

3.20.0

### Configuration file <a href="#configuration_file" id="configuration_file"></a>

The configuration file for APIM Portal is `assets\config.json`. You can see the default configuration in the example below.

```
Unresolved directive in installation-guide-portal-ui-configuration.adoc - include::https://raw.githubusercontent.com/gravitee-io/gravitee-api-management/master/gravitee-apim-portal-webui/src/assets/config.json[]
```

#### Mandatory configuration <a href="#mandatory_configuration" id="mandatory_configuration"></a>

The only mandatory value in the `config.json` file is `baseURL`, which describes where the APIM API Portal endpoint is. You must set this value so that APIM Portal can send requests to it.

### Configure the APIM Portal theme <a href="#configure_the_apim_portal_theme" id="configure_the_apim_portal_theme"></a>

You can change the default theme of APIM Portal to your own custom theme.

You customize the theme in APIM Console with the **Settings > Theme** menu option.

Whenever you change a setting, you can see the change reflected in the live preview to the right.

|   | To use the live preview, you must first configure a **Portal URL** in the Portal settings: |
| - | ------------------------------------------------------------------------------------------ |

FULLSCREEN

This button opens the preview in a new window, making it easier to edit if you have several screens.

|   | This button is only visible if you configure a Portal URL, as described above. |
| - | ------------------------------------------------------------------------------ |

RESET

This button allows you to reset the theme from the last backup.

SAVE

This button saves your theme.

ENABLED

This switch activates the theme in APIM Portal.

RESTORE TO DEFAULT THEME

This action overwrites your modifications with the theme provided by default.

#### Basic customization <a href="#basic_customization" id="basic_customization"></a>

Some basic customization options are:

Images

Add your logos. **Optional logo** is used for the home page and the footer, but you can override this to use same logo for everything by clicking **Use default logo**.

Homepage

Add your Homepage background image.

Colors

Define your primary, neutral and font colors.

Fonts

Choose your font family and sizes. Medium sizes are used by default.

#### Advanced customization <a href="#advanced_customization" id="advanced_customization"></a>

If you want to customize further, you can define the graphic properties to expose for each component.

Each component uses its own properties, but where possible, we group the properties into common variables such as the ones listed in Basic customization above.

You can see common component colors, for example, by holding your mouse over the color bubble. For other property types, if a common property is used, you can see this in the placeholder field.

### Override theme files <a href="#override_theme_files" id="override_theme_files"></a>

APIM API comes with a default theme and two default logos:

* `definition.json`
* `logo.png`
* `logo-light.png`

These files are located in the `/themes` folder of the API distribution folder.

To customize the Portal theme you can either modify these three files or specify a new folder in the `gravitee.yml` file:

```
# Portal themes
portal:
  themes:
    path: ${gravitee.home}/themes
```

By default, this configuration is commented out and the path is `${gravitee.home}/themes`

|   | <p>To help you to create your theme, you can use the editor in <strong>Settings > Theme</strong> and export it to a JSON file with the <strong>EXPORT</strong> button in the top menu. However, there are two important things to keep in mind:</p><ul><li>Images and logos cannot be changed using this method. You need to edit the two files in the distribution.</li><li>Exported themes do not have the same format as the provided <code>definition.json</code> file. You will need to make some minor edits to the exported theme.</li></ul><p><strong>Expected format</strong></p><pre><code>{
  "data": [
    {
      "name": "gv-theme",
      "css": [
        {
          "name": "--gv-theme-color-darker",
          "description": "Primary darker color",
          "type": "color",
          "default": "#383E3F",
          "value": "#383E3F"
        },
        ...
      ]
    },
    ...
  ]
}
</code></pre> |
| - | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |