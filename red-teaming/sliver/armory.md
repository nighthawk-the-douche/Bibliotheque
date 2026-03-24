# Armory

To install a specific extension (utility) we can use the `install` subcommand while specifying the name of the extension (tool/utility). If we execute only the `armory` command it will provide us with information about the current extension.

{% hint style="info" %}
To overcome a potential error with the `armory` command within the `client` component of Sliver, please run the command within the `server` component.
{% endhint %}

```bash
sliver > armory
sliver > armory install seatbelt
sliver > armory install all
```
