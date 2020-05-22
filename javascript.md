### Loops

##### Loop through Object

Key Advantage - Breakable unlike forEach

```js
for (let key in someobject) {
    if (someobject[key] == "something") {
        code = type["ID"];
        break;
    }
}
```

#### Loop through Array
for (let type of response) {
    if (type["NAME"] == blood) {
        code = type["ID"];
    }
}