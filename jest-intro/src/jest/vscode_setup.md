# Using Jest with VSCode

The main Jest plugin for VSCode is here: [Jest - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Orta.vscode-jest)

  

There is a config setting to turn off running tests automatically on save - add the following line to your `settings.json` file:

```plain
"jest.autoRun": "off",
```
  

The plugin is known to not work so well with yarn workspaces. In this case, it may be easier to simply use the CLI commands:

[Jest CLI Options · Jest (jestjs.io)](https://jestjs.io/docs/cli)
