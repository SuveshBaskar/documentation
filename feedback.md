## Feedback

### Star Rating
```js
app.sendTextMessage("How was your experience?")
app.sendRating({
    type: 'star',
    max: 10
})
```

### Scale of 1 to 10
```js
let options = [...Array(10).keys()];
options = options
.map(option => (option + ""))
.map(option => 
	Number(option) >= 8
	? ({title: option,text: option,ratingType: "good"})
	: (	Number(option) >= 7)
		? ({title: option,text: option,ratingType: "average"})
		: ({title: option,text: option})
)	

app.sendQuickReplies({
    "title": "How likely are you to recommend Yellow Messenger to your colleague or business relation?",
    "rating": true,
    options
},{hideInput:true}).then(resolve());
```