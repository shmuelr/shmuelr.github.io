---
layout:     post
title:      Jackson and Android
date:       2015-07-20 12:31:19
summary:    Using the Jackson json library with Android
categories: Jackson Android
---

Json (or javascript object notiation) is a great way to transfer data over the web or persist data to disk. It is not the only way, but it is by far the most popular due to its clear-text format and ease of use. Developers love that you can read json and understand what it represents. Contrast this to a binary protocol (such as Google's Protobuf library) which is faster and has a smaller data size, but cannot be read by humans. Readablity is the major reason why Json became the defacto standard for public api communication.

There are a couple of ways to work with json on Android. You can use the standard Java/Android libraries, Google's GSON library, or FasterXML's Jackson library. There are [discussions online]() which is the best to use and the generally accepted answer is Jackson. The native libraries are slow and cumbersome. GSON is the next best choice. It is really simple to use and a lot faster than the native libs. Jackson is (usally) the fastest one. But Jackson can be a bit confusing to setup.

####First, an overview of how Jackson works.

In your project you need to create a model of the object which the api is sending you. Json is merely a textual representation of some object. Once you have a Pojo (plain old Java object) representing the Json object you can use that with Jackson. You have to give Jackson both the Json (either as a String or a Stream) and the class that you want to build. Jackson will then create an object of that class type using the json to fill it's fields.

####Great, lets do it!

Step one is to add the Jackson library to your app's gradle file.
(replace 2.5.3 with the latest version)

{% highlight groovy lineanchors %}
compile 'com.fasterxml.jackson.core:jackson-core:2.5.3'
compile 'com.fasterxml.jackson.core:jackson-annotations:2.5.3'
compile 'com.fasterxml.jackson.core:jackson-databind:2.5.3'
{% endhighlight %}

After you've added these lines Android Studio should prompt you to 'sync gradle'. Do that. If that prompt doesn't come up then you should run Build > Clean.

Awesome, now you have Jackson added to your project.


Next lets create a Pojo for the json that we'll be using. For this example I will be using this [dictionary api](https://www.mashape.com/montanaflynn/dictionary) from Mashape. This is an example of the Json it returns for a search query 'book'.

{% highlight json lineanchors %}
{
  "definitions": [
    {
      "text": "A set of written, printed, or blank pages fastened along one side and encased between protective covers.",
      "attribution": "from The American Heritage® Dictionary of the English Language, 4th Edition"
    },
    {
      "text": "A printed or written literary work.",
      "attribution": "from The American Heritage® Dictionary of the English Language, 4th Edition"
    },
    {
      "text": "throw the book at  To reprimand or punish severely.",
      "attribution": "from The American Heritage® Dictionary of the English Language, 4th Edition"
    }
  ]
}
{% endhighlight %}

We can see there are two Json objects here. The first is the Response object and that contains and array of Definition objects.

#### Building the Java models

We need Java models for these objects. To do that we head over to one of my favorite websites, [JsonSchema2Pojo](http://www.jsonschema2pojo.org/). Copy the Json into the input box, set the 'Source Type' to Json, and hit 'Generate'.

{% highlight java lineanchors %}

package com.example;

import java.util.HashMap;
import java.util.Map;
import javax.annotation.Generated;
import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;

@JsonInclude(JsonInclude.Include.NON_NULL)
@Generated("org.jsonschema2pojo")
@JsonPropertyOrder({
"text",
"attribution"
})
public class Definition {

@JsonProperty("text")
private String text;
@JsonProperty("attribution")
private String attribution;
@JsonIgnore
private Map<String, Object> additionalProperties = new HashMap<String, Object>();

/**
* 
* @return
* The text
*/
@JsonProperty("text")
public String getText() {
return text;
}

/**
* 
* @param text
* The text
*/
@JsonProperty("text")
public void setText(String text) {
this.text = text;
}

/**
* 
* @return
* The attribution
*/
@JsonProperty("attribution")
public String getAttribution() {
return attribution;
}

/**
* 
* @param attribution
* The attribution
*/
@JsonProperty("attribution")
public void setAttribution(String attribution) {
this.attribution = attribution;
}

@JsonAnyGetter
public Map<String, Object> getAdditionalProperties() {
return this.additionalProperties;
}

@JsonAnySetter
public void setAdditionalProperty(String name, Object value) {
this.additionalProperties.put(name, value);
}

}
{% endhighlight %}

{% highlight java lineanchors %}

package com.example;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import javax.annotation.Generated;
import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;

@JsonInclude(JsonInclude.Include.NON_NULL)
@Generated("org.jsonschema2pojo")
@JsonPropertyOrder({
"definitions"
})
public class Response {

@JsonProperty("definitions")
private List<Definition> definitions = new ArrayList<Definition>();
@JsonIgnore
private Map<String, Object> additionalProperties = new HashMap<String, Object>();

/**
* 
* @return
* The definitions
*/
@JsonProperty("definitions")
public List<Definition> getDefinitions() {
return definitions;
}

/**
* 
* @param definitions
* The definitions
*/
@JsonProperty("definitions")
public void setDefinitions(List<Definition> definitions) {
this.definitions = definitions;
}

@JsonAnyGetter
public Map<String, Object> getAdditionalProperties() {
return this.additionalProperties;
}

@JsonAnySetter
public void setAdditionalProperty(String name, Object value) {
this.additionalProperties.put(name, value);
}

}
{% endhighlight %}

Boom. We now have our Java Pojo models. Add them to your project (I like to create a models package and put all my model objects there.)

#### Create the actual Java objects from the Json

In your project write code to get the Json. You can do this with HttpURLConnection, OkHttp, or just loading a String. Once you have the Json Stream or Json String these are the lines that do the magic:

{% highlight java lineanchors %}
ObjectMapper objectMapper = new ObjectMapper();
Response response = objectMapper.readValue(okhttpResponse.body().byteStream(), Response.class);
{% endhighlight %}

We are creating a Jackson ObjectMapper and providing it with our input (in this case I used a Stream but you can replace that with a String) and the class that we want to build. Jackson handles everything else.


That's it. Now you have a Pojo for the server's response. You can use it in your Java code wherever needed.
