## Agent-Transfer

### Transfer to live agent
	
Live agent transfer is useful when the user needs extra help with an issue. The process starts by raising a ticket. For creating a ticket, app.raiseTicket() method is invoked with various parameters.

##### Syntax for app.raiseTicket() method:
```js 
app.raiseTicket({
   issue: “Issue String”,
   contact: {
       name: app.context.steps.name || "SAMPLE NAME"
   },
   priority: 'MEDIUM',
   manualAssignment: false,
}).then(function (ticketData) {
  if (ticketData && ticketData.assignedTo) {
  // Ticket generated
  } else {
  // Ticket not generated
  }
})
```

- IssueString: Issue string helps the agent to get an idea regarding the context or issue faced by the user
- Contact: This is an object of user details collected from the user and passed for the agent’s reference. The object can contain name, email, phone number and so on.
- ticketData: Once the ticket is generated, the ticketData will contain data regarding the ticket generated. It can be the name of the agent to whom the ticket is assigned to, the ticket number and so on. 

If the ticket is not created, the error message is sent in the ticketData.

There are 2 types of ticket creation:

1 Online ticket generation
2 Offline ticket generation


### Online ticket generation:

Online tickets are generated when the customer service agents are online. When the online ticket is generated, the bot will be paused using the command app.pauseBot() to allow the user to converse with the agent.

##### Code for online ticket generation:

```js
app.sendTextMessage(app.renderMessage('pleaseWait', {}, "Please wait while I contact my support center...")).then(() => {
   let issueString = "User Query"
   app.raiseTicket({
       issue: issueString,
       contact: {
           name: app.context.steps.name || "SAMPLE NAME"
       },
       priority: 'MEDIUM',
       manualAssignment: false,
   }).then(function (ticketData) {
       app.log(ticketData, "TICKET DATA")
       if (ticketData && ticketData.assignedTo) {
           app.pauseBot();
           app.sendTextMessage(app.renderMessage('patience', {}, "Thank you for your patience, you are now connected with our agent.")).then(() => {
            return resolve();
           });
       } else {
           app.sendTextMessage(app.renderMessage('unavailable', {}, "Our agents are currently unavailable. We will get back to you as soon as possible."))
           return resolve()
       }
   })
})
```

### Offline ticket generation:

Offline tickets are generated when the customer service agents are either busy with serving other users or offline when taking a break after answering the queries of other users.

While generating an offline ticket, 2 main parameters needs to be sent to the app.raiseTicket() method. Offline tickets can be generated if the online tickets are not generated.

``` 
  manualAssignment: true,
  assignedTo: 'emailId'
```
- manualAssignment: This flag should be set to true
- assignedTo: The email id of the agent to whom the tickets has to be assigned.
 

##### Code for offline ticket generation:
```js
app.raiseTicket({
  issue: 'Customer Query',
  contact: {
    name: profileName,
    phone: profileName,
    email: "",
  },
  priority: 'MEDIUM',
  manualAssignment: true,
  assignedTo: 'emailId of agent'
}).then((ticketData) => {
  if (ticketData && ticketData.assignedTo) {
    app.pauseBot();
    app.sendTextMessage(app.renderMessage('patience', {}, "Thank you for your patience, you are now connected with our agent.")).then(() => {
      return resolve();
    });
  } else {
    app.sendTextMessage(app.renderMessage('unavailable', {}, "Our agents are currently unavailable. We will get back to you as soon as possible."))
    return resolve()
  }
})

```
