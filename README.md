# How-to: Set up React Native with F# and Fable

This is a step-by-step guide to setup a React Native project with F# and Fable (with Elmish). This lets you write React Native apps targeting iOS and Android almost completely through F#! Sample files for getting started can be found within this repository.

## Requirements
- React Native
- Watchman
- Node.js
- .NET Core >= 3.0
- paket
- yarn (or npm, but i use yarn here)

# Setup a new React Native project

This how-to will not go into detail on how to install React Native and how React Native works. When you have React Native reacy you can reate a new project by running `react-native init <project name>`

You should test that your basic React Native project compiles/runs before moving further.

`react-native run-ios` or `react-native run-android`

# Setup the F# project
Create a folder to hold your F# project and add a `.fsproj` file with a simple `.fs` file. In this example i will create a `src` folder in the project root directory which contains the files `App.fsproj` and `App.fs`.

## `App.fsproj`
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="App.fs" />
  </ItemGroup>
  <Import Project="..\.paket\Paket.Restore.targets" />
</Project>
```

## `App.fs`
```fsharp
module App

open Elmish
open Elmish.React
open Elmish.ReactNative
open Fable.ReactNative

// A very simple app which increments a number when you press a button

type Model = {
    Counter : int
}

type Message =
    | Increment 

let init () = { Counter = 0 }, Cmd.none

let update msg model =
    match msg  with
    | Increment -> 
        { model with Counter = model.Counter + 1 }, Cmd.none 

module R = Fable.ReactNative.Helpers
module P = Fable.ReactNative.Props
open Fable.ReactNative.Props

let view model dispatch =
    R.view [
        P.ViewProperties.Style [ 
            P.FlexStyle.Flex 1.0 
            P.FlexStyle.JustifyContent JustifyContent.Center
            P.BackgroundColor "#131313" ]
    ] [
        
        
        R.text [
            P.TextProperties.Style [
                P.Color "#ffffff"
            ]
        ] "Press me"
        |> R.touchableHighlightWithChild [
            P.TouchableHighlightProperties.Style [
                P.FlexStyle.Padding ( R.dip 10. )
            ]
            P.TouchableHighlightProperties.UnderlayColor "#f6f6f6"
            OnPress ( fun _ -> dispatch Increment ) 
        ]

        R.text [
            P.TextProperties.Style [
                
                P.Color "#ffffff"
                P.FontSize 30.
                P.TextAlign P.TextAlignment.Center
                
            ]
        ] ( string model.Counter )
    ]

Program.mkProgram init update view
|> Program.withConsoleTrace
|> Program.withReactNative "Your project name" // CHANGE ME
|> Program.run
```

IMPORTANT: Feed the name of your project in `Program.withReactNative` (the same you used for `react-native init` )

# Install Nuget packages

1. Create a `paket.dependencies` file in the root folder
2. Add the following Nuget packages to the list:
    ```
    source https://www.nuget.org/api/v2
    storage: none

    nuget Fable.Core 
    nuget Fable.React
    nuget Fable.Elmish 
    nuget Fable.Elmish.React
    nuget Fable.React.Native
    ```
3. Add a `paket.references` file in your F# project folder (the one named `src` in this example). Add your Nuget packages to the `paket.references` file. 
    ```
    Fable.Core
    Fable.React
    Fable.Elmish
    Fable.Elmish.React
    Fable.React.Native
    ```

4. Run `paket install` in your root folder.
    - Can be installed as a global tool: `dotnet tool install paket -g`

# Install npm-packages

Install the [fable-splitter](https://www.npmjs.com/package/fable-splitter) and [fable-compiler](https://www.npmjs.com/package/fable-compiler) npm-modules. See the documentation of these for further info.

`yarn add --dev fable-splitter fable-compiler` 

`yarn add buffer`

`yarn add --dev @babel/preset-env`

Go ahead and delete your `babel.config.js` file.

You can now compile your F# project to Javascript by simply running `yarn fable-splitter src/App.fsproj -o out`
(Note the `-o` parameter specifying the output folder to dump the .js files) 

You can provide the `fable-splitter` with a config file for simpler configuration. For example:
```javascript
module.exports = {
    entry: "src/App.fsproj",
    outDir: "out",
    babel: {
      presets: [["@babel/preset-env", { modules: "commonjs" }]],
      sourceMaps: false,
    },
    // The `onCompiled` hook (optional) is raised after each compilation
    onCompiled() {
        console.log("Compilation finished!")
    }
  }
```
`fable-splitter -c splitter.config.js`

Note the `entry` value should be the path to whatever you called your `.fsproj` file and the `out` value should be your output folder. This sample configuration will compile your F# project located in `./src/App.fsproj` into JavaScript dumped into the `./out` folder.

#### Tips:
```json
"build": "fable-splitter -c splitter.config.js --define RELEASE",
"debug": "fable-splitter -c splitter.config.js --define DEBUG",
"watch": "fable-splitter -c splitter.config.js -w --define DEBUG"
```
Add the above JSON to the `scripts` section of the `packages.json` file and simply call `yarn build` to compile. Run `yarn watch` in order to watch for changes and enable hot-reloading as you change your F# code.

# Importing the generated JavaScript
Now you can compile your F# code to JavaScript and dump it to a folder. In the sample `splitter.config.js` file we specified that the generated JavaScript is to be dumped to the `out` folder.

1. Delete your default `App.js` file in the root directory.
2. Update your `index.js` file:
    ```js
    /**
    * @format
    */

    import { AppRegistry } from 'react-native';
    import * as App from './out/App';
    import { name as appName } from './app.json';
    ```

# You're good to go! 
1. Compile F# to JavaScript and watch for changes
    - `yarn fable-splitter -c splitter.config.js -w --define DEBUG`
    - or `yarn watch` if you altered the `scripts` section of `packages.json`
2. Run app
    - `react-native run-ios|android`
3. Watch as the app running updates as your F# code is changed. Enjoy!
