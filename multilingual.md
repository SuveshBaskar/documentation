### Custom Entity Recognizer
```js
app.customEntityRecognizer = function (prediction) {
    return new Promise(resolve => {
        if (app.data.message && app.data.message == "welcome") {
            prediction.confidence = 1;
            prediction.intent = "start";
        }
        return resolve(prediction);
    })
}
```

### Bot Options
```js
app.CUSTOM_BOT_OPTIONS = {
    minConfidence: 0.80,
    targetLanguage: "en",
    i18n: true,
    excludeParamsForSwitching: [],
}
```

### Default Voice Options
```js
app.VOICE_OPTIONS = {
    capture_dtmf: false,
    tts_engine: "wavenet",
    text_type: "ssml",
    speed: 0.9,
    disconnect: false
}
```

### Translator Function
```js
app.TRANSALTE_HELPER = async (message) => {
    if (message) {
        try {
            let result = await app.detectLanguageAndTranslate(message, "en");
            let detectedLanguage = result.data.translations[0].detectedSourceLanguage;
            message = result.data.translations[0].translatedText.trim();
        } catch (error) {/* DO NOTHING */}
        return message.trim();
    } else {
        return message;
    }
}
```

### Language Helper
```js
app.GET_LANGUAGE_IN_MEMORY = () => {
    return app.memory.get("language")
        .then(language => {
            return language
        })
        .catch(error => {
            return app.DEFAULT_LANGUAGE
        })
}
```

### Bot Start Helper
```js
app.START_BOT = async () => {
    let language = await app.GET_LANGUAGE_IN_MEMORY();
    app.log(language, "[LANG MEM]"); // Logging
    if (language) {
        if (language == "en") {
            return app.start(app.CUSTOM_BOT_OPTIONS);
        } else {
            app.data.message = await app.TRANSALTE_HELPER(app.data.message);
            app.log(app.data.message, "[TRANSLATED MESSAGE]") // Logging
            app.CUSTOM_BOT_OPTIONS.targetLanguage = "hi";
            return app.start(app.CUSTOM_BOT_OPTIONS);
        }
    } else {
        await app.memory.set("language", app.DEFAULT_LANGUAGE);
        return app.start(app.CUSTOM_BOT_OPTIONS);
    }
}
```

### Welcome Helper - IVR
```js
if (app.data && app.data.message == "welcome") {
    try {
        app.getContext()
            .then((context) => {
                app.clearContext();
            })
    } catch (error) { }

    try {
        app.memory.delete("language")
    } catch (error) { }

    app.log("BOT STARTED", "[INFO]")

    app.START_BOT();
} else {
    app.START_BOT();
}
```