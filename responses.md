## Responses

#### Text Message

```js
app.sendTextMessage('message')
```

#### Quick Replies

##### Default

```js
app.sendQuickReplies({ 
	title: 'VALUE',
	options: [
		{
			title: 'OPTION',
			text: 'OPTION'
		},
	]
});
```

##### Short hand

```js
let options = ["Option1","Option2","Option3"];
options = options.map(option => ({title: option, text: option}));
app.sendQuickReplies({ 
   title: 'VALUE',
   options,
});
```

##### Options
```js
let options = ["Option1","Option2","Option3"];
options = options.map(option => ({title: option, text: option}));
app.sendQuickReplies(
  { 
   title: 'VALUE',
   options,
},{hideInput:true});
```

##### Multi Select
```js
let options = ["Option1","Option2","Option3"];
options = options.map(option => ({title: option, text: option}));
app.sendQuickReplies({
    title: "Please select option(s)",
    multiSelect: true,
    options ,
    cancelButton: {
        title: "Cancel",
        text: "Cancel"
    },
    submitButtonText: "Submit"
})
```

#### Cards
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

#### Send Table in Cards
```js
app.sendCards(
 {
   title: 'text',
   image:'image_link',
   text: 'The details about our organization is specified here',
   table: [
     {
         title: 'text',
         value: 'text'
     },
     {
         title: 'text',
         value: 'text'
     }
   ]
 }
);
```

#### Webview
```js
app.sendWebView('Please select your date of birth', 'https://yellow.chat/datePicker', 350, { hideInput: true }).then(() => {
  return resolve();
})
```

Date Picker - https://yellow.chat/datePicker
Feedback - https://yellow.chat/clixcapital