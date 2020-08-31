
#### Workshop Terminals
Two terminals are included in this workshop, one terminal is used to run a server and the other terminal to interact with the server and also to execute commands.

#### Workshop Code Editor
The workshop features a built in code editor you can use by pressing the `Editor` tab button. Pressing the refresh button in the workshop's UI can help the editor load when switching tabs. The files in this editor automatically save.

### Environment Build Tools
You can build your app with `Maven` and `Gradle` as they are both included in the environment. Only the `Maven` commands are in the workshop for brevity, but you can use `Gradle` on your own if you prefer. 

#### Workshop Command Execution
To progress through the workshop you execute commands by clicking the running man on the commands in the text, they are configured to run in the appropriate terminals. Try this when you clone the GitHub repo in the next step.

During this workshop we will be using the command line as well as the embedded editor. The editor takes a few moments to start up and be ready, so select the **Editor** tab now to display it, or click on the action block below.

```dashboard:open-dashboard
name: Editor
```

The workshop uses these action blocks for various purposes. Anytime you see such a block with an icon on the right hand side, you can click on it and it will perform the listed action for you.

#### Clone Github Repo

Clone the source repository for this guide using Git on the 1st terminal: 

```execute-all
git clone https://github.com/spring-guides/gs-gateway.git
```

```execute-all
cd ./gs-gateway/initial/
```