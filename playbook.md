# Playbook 

This will contain all the codes that are frequently used and is commonly used across all bots

### Basic Inputs Helper
```js
const { message, event } = app.data;
const { steps, intent } = app.context;
const step = app.paramExpected;
```

### Parsing String
```js
try {
    let user = await app.memory.get("user-001");
    user = typeof user == "string" ? JSON.parse(user) : user;
} catch (error) {/* DO NOTHING */ }
```


### Delete Memory
```js
try {
    await app.memory.delete("validator-invalid-count-004");
} catch (error) {/* DO NOTHING */ }
```

### Test Messages
```js
if(app.data.message == "@test"){
    app.log("LOG:001","[TEST]");
    return app.sendTextMessage("TEST:001");
}

if(app.data.message == "@trig"){
    app.log("LOG:001","[TEST]");
    return app.triggerIntent("intent01");
    // return app.triggerIntent("intent02");
    // return app.triggerIntent("intent03");
}
```