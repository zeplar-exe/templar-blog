---
layout: post
title:  "States of the Country - Devlog #1 \"Configuration\""
date:   2022-12-20 8:23:01 -0800
categories: devlog
---

One big part of my game design philosophy is the ability for the user to do whatever they want. Not just in the game, but in how the game itself functions. Of which is easily achieved by external configuration files. What do I mean by that? Literally just a couple hundred JSON files shipped with the executable.

### But why?

* It's easy to change on my part. 
* It's easy to change on the player's part. 
* It's easy to implement modding on my part. 
* It's easy to make on my part.

In all, only having to create/edit a JSON file is much easier than hard-coding that new functionality. Plus, you can use a [JSON schema](https://json-schema.org/) to ensure the format of your file.

### But how?

[Json.NET](https://www.newtonsoft.com/json) makes this trivially easy. Just define a class with the expected data, its keys, and any special converters, then just run and boom. 10 lines of code to deserialize everything.

Take this json representation of a pop-up event;

```json
{
    "id": "USA.social_unrest",
    "type": "city_event",
    "name": "Social Unrest",
    "body": "Citizens of {city.name} are dissatisfied with the current government.",
    "effects": {
        "add_social_stability": "-5"
    },
    "options": [
        {
            "text": "Initiate propaganda efforts.",
            "effects": {
                "select_weighted": {
                    "options": [
                        {
                            "weight": "50w",
                            "tooltip": "The propaganda further enrages the population.",
                            "effects": {
                                "add_social_stability": "-5"
                            }
                        },
                        {
                            "weight": "50w",
                            "tooltip": "The population is placated.",
                            "effects": {
                                "add_social_stability": "5"
                            }
                        }
                    ]
                }
            }
        },
        {
            "text": "Leave it be."
        }
    ]
}
```

> It should be noted that my main inspiration of this format comes from Paradox games like CK3 and EU4. Though my bitter distaste of their implementation caused me to use JSON like this. You can read more about that in a post soon to come...

The only real setup I have to do in C# is the following;

```cs
public class EventData
{
    [JsonProperty("id")] public string Id { get; } = DataConstants.MissingId;
    [JsonProperty("type")] public EventType Type { get; } = EventType.None;
    [JsonProperty("title")] public string Title { get; } = DataConstants.MissingString;
    [JsonProperty("body")] public string Body { get; } = DataConstants.MissingString;
    [JsonProperty("effects")] public EffectReferenceContainer Effects { get; } = new();
    [JsonProperty("options")] public List<EventOption> Options { get; } = new();
}
```

That is it. No complex serialization code, not too many manual instructions. Newtonsoft does everything for me.

### So why does this matter?

With the ever-expanding ideas I have for this project, the ability to quickly and easily make changes is essential. The flexibility this provides is so simple yet so vast, that plenty of technical debt and spaghetti has been avoided. 

In all, I highly encourage game developers (and software developers too) to take on a similar approach when designing such data-intensive applications. Not only will it save you time, but also on would-be difficulties that come down the line.