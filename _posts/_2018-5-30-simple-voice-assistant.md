---
layout: post
title: Simple Voice Assistant 
subtitle: Build a Siri-like with Text-to-Speech and Speech-to-Text
date:   2018-5-30
background: '/img/assistant.jpg'
---
How do your users interact with your app? Keyboard input to fill content? Tap tap tap to select what they want? Yes, same here. All of my apps are developed in this traditional way. 

In my latest app, [Coinhako](https://itunes.apple.com/us/app/coinhako-wallet/id1137855704?mt=8), I add a simple voice assistant to help users to command what they want. Just the first version, so support only 4 basic actions, buy/sell/send/receive cryptocurrency. 

Today, I show you how to make an assistant like this with scalable and easy to maintain. 

The idea to this is: 

- You have a controller like an assistant. The Assistant has Speaker (Text-to-Speech) and Recorder (Speech-to-Text). 
- When you activate the Assistant, it asks you about your command, and start recording. 
- The recorder records everything you say and pass the content to an analyser. 
- The Analyser analyzes and picks the SpeechOrder base on your command. 
- The SpeechOrder responses to the Assistant the next step and next message to speak. 
- After step by step from The SpeechOrder, it decides what destination needs to display. 

What I show you in the demo is the final design after a code struggle. With this design, you can easily add new support action with simple step: add new command, add new vocabulary, and add a new SpeechOrder. 

Let's do it. 

### Preparation

You can download the start project [here](https://drive.google.com/file/d/1GrNJQ9oV1JzjreHvFbNCxQqFSeQ31ciA/view?usp=sharing).


### Add the assistant class 

