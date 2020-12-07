---
type: Blog
title: Alexa skill(s) backed by Java Resource(s)
summary: I wanted to play around with resource based Alexa skills - which doubled as playing with reflection and annotations.
categories: [Blog]
tags: [Java, Alexa]
---

I wanted to start playing around with Alexa skills, I had a view ideas but no experience and needed a good starting place.  Figured I’d hit two birds with one stone by creating a base skill handler to take advantage of some common features across all the skills that I wanted to work towards.  I know that Alexa works with locales and I wanted my skills to be available across the board; or to be extendable easily for others.

Creating a simple set of abstract Handlers and Skills that could easily lookup message content for different locales using Java ResourceBundles.  I figured that starting off with localed responses would be good, even though after looking into how requests work the Alexa Skill would need to have custom interaction models for each language.  Throughout this project, the code will be available at Github.  If at any point, anyone out there reading this, sees that I zigged when I should have zagged, please let me know.

## Project Overview

Following a number of the tutorials on the Alexa SDK [https://developer.amazon.com/docs/alexa-skills-kit-sdk-for-java/overview.html](https://developer.amazon.com/docs/alexa-skills-kit-sdk-for-java/overview.html).   When I started the project I was hadn't gotten too far into Node, so Java seemed to be a decent way to get started; since then I've been doing more JavaScript development and gotten a little more comfortable with Node and will have slowly let this version of the project go.

## AlexaLocaleStreamHandler

Creates the Alexa skill using the SDK, each of the handlers provided can be hard coded or can be implementations of the LocaledRequestHandler.  Getting started the skill is created as documented:

```java
public class AlexaLocaleStreamHandler extends SkillStreamHandler {
  private static final String ENV_APP_ID = "alexa.app.id";

  private static Skill getSkill() {
    return new CustomSkillBuilder()
      .withSkillId(System.getenv(ENV_APP_ID))
      .addRequestHandlers(new LaunchRequestHandler(),
          new HelpRequestHandler(),
          new CancelOrStopRequestHandler())
      .addExceptionHandlers(new UnhandledSkillExceptionHandler())
      .build();
  }
	
  public AlexaLocaleStreamHandler() {
    super(getSkill());
  }	
}
```

## LocaledRequestHandler

The LocaledRequestHandler is responsible for loading the supplied `ResourceBundle` when the annotation `@LocaleResourceBase` is provided on the class.  The following `HelloWorldHandler` will load and use the `messages/hello-world.properties` file (using standard resource locale selectors).

```java
@LocaleResourceBase("messages/hello-world")
class HelloWorldHandler extends LocaledRequestHandler {
  @Override
  protected Optional<Response> handleRequest(HandlerInput input, ResourceBundle rb) {
    String speech = getMessage(rb, "welcome", "Welcome to the Alexa locale skill.");
    return input.getResponseBuilder()
      .withSpeech(speech)
      .withReprompt(speech)
      .withSimpleCard("Hello", speech)
      .build();
  }
}
```

If no value is specified for the `@LocaleResourceBase` annotation, then the properties file used should exist within the same class path, for example the default file would be `kjd.alexa.locale.handler.HelloWorldHandler.properties`:

```
welcome=Welcome to the Alexa locale skill.
```

## Update 

This project kind of fell by the wayside as I slowly started getting more into Node development at work and in my free time.  