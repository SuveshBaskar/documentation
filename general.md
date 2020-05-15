## Links for Bot

### Bot Test Link
```text
https://app.yellowmessenger.com/api/chat/demo/botId
```

### Bot PWA Link
```text
https://app.yellowmessenger.com/pwa/mobile/botId
```

### Bot PWA Link (Full Screen)
```text
https://app.yellowmessenger.com/pwa/live/botId?fullScreen=true
```

## App Object
```js
app.log(app.source) // Message Source
app.log(app.profile) // Sender Profile
```

## Responses

#### Text Message
```js
app.sendTextMessage('message')
```

#### Quick Replies
```js
app.sendQuickReplies({ 
       title: 'text',
       options: [
         {
           title: 'text',
           text: 'text'
         },
         {
           title: 'text',
           text: 'text',
         }
       ]
     });
```

#### Send Cards
```js
app.sendCards(
 {
   title: 'text',
   image:'image_link',
   text: 'The details about our organization is specified here',
   actions: [
     {
         title: 'Know More',
         text: 'know more'
     }
   ]
 }
);
```


#### Localization & app.renderMessage()
```js
message {{code}} // In Localization

app.sendTextMessage(app.renderMessage('name_in_localization',{code:code_value},default_message)).then(resolve);// Inside function
```

## Custom Handler : resolve({success:false})
```
return resolve({
            success:false,
            customHandler:()=>{
                return app.sendQuickReplies({
                    title:`Sorry! I could not understand your choice. Could you please choose a language from the following?`,
                    options:[
                        {
                            title: `English`,
                            text:`English`
                        },
                        {
                            title:`Hindi`,
                            text: `Hindi`
                        }
                    ]
                });
            }
        });
```

## Send Email
```
app.sendEmail(email_id, 'Subject','Message', {Attachments if any} ,'bot@yellowmessenger.com','')

```
## Remove unnecessary words while getting user name
```
let removeUselessWords = function(txt) {
  let uselessWordsArray = 
      [
        'a', 'at', 'be', 'can', 'cant', 'could', 'couldnt', 
        'do', 'does', 'how', 'i', 'in', 'is', 'many', 'much', 'of', 
        'on', 'or', 'should', 'shouldnt', 'so', 'such', 'the', 
        'them', 'they', 'to', 'us',  'we', 'what', 'who', 'why', 
        'with', 'wont', 'would', 'wouldnt', 'you','my','name','is','you','can','call','me','I','am','yeah','sure'
      ];
    
  let expStr = uselessWordsArray.join('|');
  return txt.replace(new RegExp('\\b(' + expStr + ')\\b', 'gi'), ' ')
            .replace(/\s{2,}/g, ' ');
}

let str = 'My name is Ashish';
  
app.log(removeUselessWords(str));
```
## Time and Date
#### Change time and date to desired formats
```
app.dateMsg = (val) => {
   let date = new Date(val);
   let weekday = new Array('Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday');
   let months = new Array('January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December');
   let dayOfWeek = weekday[date.getDay()];
   let dateNum = date.getDate();
   let curMonth = months[date.getMonth()];
   let curHour = date.getHours() > 12 ? date.getHours() - 12 : (date.getHours() < 10 ? '0' + date.getHours() : date.getHours());
   let curMinute = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes();
   let curMeridiem = date.getHours() > 12 ? 'PM' : 'AM';
   return `${dayOfWeek} ${dateNum} ${curMonth} ${curHour}:${curMinute}${curMeridiem}`;
}
```
## Elastic Search Queries - Data Store

#### Get data that matches a particular value in a column
```
 app.dataStore.search({
        table: table_name,
        body: {
            'query': {
                'match': {
                    column_name: {
                        'query': search_term,
                        'fuzziness': 1
                    }
                }
            }
        }
    }).then((records) => {
        if (records.hits && records.hits.hits.length > 0) {
              app.log(records.hits.hits[0], 'Record');
      }
    })
```
#### Multi-match Query

```
{
  'query': {
    'multi_match' : {
      'query':    'this is a test', 
      'fields': [ 'subject', 'message' ] 
    }
  }
}
```

## Druid Queries

#### For Custom Event

```
{
        'queryType' : 'timeseries',
        'dataSource' : 'metrics',
        'granularity' : 'all',
        'aggregations' : [ 
             {
            'type': 'doubleSum',
            'name': 'sum',
            'fieldName': 'value_sum'
        }
        ],
        'intervals' : [ 
            '2019-06-28T00:39:42.231Z/2019-07-05T23:39:42.231Z'
        ],
        'metric' : 'sum',
        'filter' : {
            'type' : 'and',
            'fields' : [ 
                {
                    'type' : 'selector',
                    'dimension' : 'aevent',
                    'value' : '<event-name>'
                },{
                  'type' : 'selector',
                  'dimension' : 'metric',
                  'value' : 'events.bots.custom'
              }
                
            ]
        }
    }
```

## Working with API

#### Call API
```
function api_call() {
  var options = {
    method: 'POST',
    url: 'api_endpoint',
    headers:
    {
      'cache-control': 'no-cache',
      'Authorization': 'accessToken',
      'Content-Type': 'application/json'
    },
    body:
    {
    },
    json: true
  };

  app.request.request(options, function (error, response, body) {
    if (error) {
      reject(new Error(error));
    } else {
      app.log(response,'RESPONSE');
      app.log(body,'BODY');
    }
}
```
## In Multilingual Bot 

#### To check translated message
```
app.customEntityRecognizer = function (prediction) {
    return new Promise((resolve) => {
        app.log(prediction.text, 'TRANSLATED TEXT`);
        return resolve();
    }, (error) => {
        return resolve();
    });
}
```

#### To translate the user message to English
```js
const detectLanguage = () => {
    return new Promise((resolve) => {
        var options = {
            method: 'POST',
            url: 'https://translation.googleapis.com/language/translate/v2',
            qs: { key: 'API_KEY' },
            headers:
            {
                'cache-control': 'no-cache',
                'content-type': 'application/json; charset=utf-8'
            },
            json: true,
            body: {
                q: app.data.message,
                target: 'en',
                format: 'text'
            }
        };
        app.request.request(options, function (error, response, body) {
            if (body && body.data && body.data.translations && body.data.translations.length > 0) {
                app.data.message = body.data.translations[0].translatedText;
                resolve('en');
            }
        });
    });
}

const startBot = () => {
    return detectLanguage().then((lang) => {
        return app.start({
            minConfidence: 0.8,
            contextConfidence: 0.85,
            invalidCount: 2,
            targetLanguage: 'en',
            i18n: true,
            excludeParamsForSwitching: ['know_more', 'variants'],
            translateInputMessages: false,
            onInvalidCountExceeded: () => {
                app.clearContext();
            }
        });
    });
}


return startBot();
```


# Whatsapp

## Bot Link
```text
https://api.whatsapp.com/send?phone=917829163201&text=Talk%20to%20p%20b
```

## Send Image
```js
app.sendImage(
  'https://cdn.yellowmessenger.com/s5JDgUoNOCpv1571732866794.png', 
  {
    mime: 'image/png',
    caption: '\n\nA visual on P1M and current month margins made-(Brand+TRP+Visibility+Corporate)'
  }
)
```

## Send Document
```js
app.sendDocument(
  'https://corporate.nbbonline.com/corp/web/L001/corporate/jsp/common/NBB_Business_Online_Banking_FORM_v1.pdf',
  { 
    mime: 'application/pdf', 
    filename: 'Application Form' 
  }
)
```

## Handy Snippets

## Greeting
```js

  let date = new Date()
  date.setHours((date.getHours()+5),(date.getMinutes()+30)) //Convert from UTC to IST
  let hours = date.getHours() 
  let greet = (hour > 16) 
              ? "Good Evening" 
              : (hour >= 12) 
                ? "Good Afternoon" 
                : "Good Morning"
  app.sendTextMessage(`${greet}, How can I help you today?`)
```



### For Yes or No response
```js
if (app.prediction.global_model && (app.prediction.global_model.intent === 'yes' || app.prediction.global_model.intent === 'okay') && app.prediction.global_model.confidence >= 0.80) {
  app.setStep('confirmation', 'YES').then(() => {
    resolve();
  })
} else if (app.prediction.global_model && app.prediction.global_model.intent === 'no' && app.prediction.global_model.confidence >= 0.80) {
   app.setStep('confirmation', 'NO').then(() => {
    resolve()
  })
}
```

### Check if the user message triggers someother journey
```js
if(app.prediction && app.prediction.intent && app.prediction.confidence > 0.8){
  return app.triggerIntent(app.prediction.intent)
}
```

## Render Message
```text
Use {{{}}} for unicodes in Localization
```

## Send Event
```js

//{"event_code":"custom-event","data":{"event":"close-bot","data":{}}}
app.sendEvent({
    code: "close-bot",
    data: {}
})
```
