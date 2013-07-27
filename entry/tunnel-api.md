{{$ meta }}
title: using third party APIs without JSONP
created: 2013-06-16
public: yes
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
disqusid: 9ed73595-c85d-4f9a-ac0e-9c6ba7ad8534
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

I'm excited to announce the new [Backlift Tunnel API](http://backlift.github.io/docs/tunnel.html#tunnel-api). The goal of this feature is to make it easier to incorporate third party APIs into your Backlift website. The tunnel API eliminates the need for complex CORS or JSONP work-arounds to the [same origin policy](http://en.wikipedia.org/wiki/Same_origin_policy), and it lets you store your secret keys privately on the server, rather than exposing them in your javascript code. It does this by providing a way to configure tunnels, or proxy endpoints, and attach headers, parameters and authorization details, that will be used to send requests from the server.

{{$ break }}

<!--
Basically with the Tunnel API you can create endpoints in your config.yml file and attach headers, auth and other parameters to those endpoints. For example:

    tunnels:
        someothersite:
            base_url: 'https://someother.com/api/'
            auth:
                basic_user: 'laurihy'
                basic_pass: 'mysecretpassword'
            
Then when you send requests to a tunnel endpoint, your requests will be proxied to the base url that you configured, along with any auth, parameters or headers that you added. For example, given the above configuration, a request to `/backlift/tunnel/someothersite/foo` would result in the following request:

    $.get('/backlift/tunnel/someothersite/foo')
    // -> GET https://someother.com/api/foo
    //        Headers: 'Authorization: Basic QWxhZGRp… 
-->

For a more in-depth explanation of the Tunnel API, please check out our docs here: [http://backlift.github.io/docs/tunnel.html](http://backlift.github.io/docs/tunnel.html#tunnel-api). To see an example of the kind of limitation that the Tunnel API helps overcome, check out [this stackoverflow question](http://stackoverflow.com/questions/3350778/modify-http-headers-for-a-jsonp-request).

In the rest of this post I'll provide a quick example of how you can use the Tunnel API to do something awesome. It's currently 5pm, and I'm getting hungry. I'm going to build a Backlift app that I can use to find pizza joints that deliver to my address using the [ordr.in API](https://hackfood.ordr.in). Let's do it!

## The setup
First things first, I'll create a new app on Backlift. I'm going to clone the "Hello World" app as a starting point, and call my app "pizzafinder". After a few seconds my app will be created, and I'll be able to open up my pizzafinder folder with a text editor. 

<img src="/images/pizzafindercreated.png">

I also need to create a new app on ordr.in. After I've registered a developer account, I can click the "create new app" link on my ordr.in dashboard and enter the details for the app I'll be creating.

<img src="/images/createnewapp.png">

Ordr.in will now generated a pair of keys for my app that I can use for future requests to its API.

## Finding restaurants nearby

I'm going to build an app that fetches a list of pizza joints that deliver to my current address. I'll start off by generating a simple HTML page with an address form and a submit button that will fetch a list of nearby restaurants. Here's my new `index.html` file:

**index.html**

    <!DOCTYPE html>
    <html>
      <head>
        <title>PIZZA FINDER!!!!</title>
        {{$styles}}
      </head>
      <body>
        <div class="container">
          <form>
            <fieldset>
              <label>address</label> <input name="address" type="text"/>
              <label>city</label> <input name="city" type="text"/>
              <label>zip</label> <input name="zip" type="text"/>
            </fieldset>
            <input type="submit" class="btn"/>
          </form>
          <ul></ul>
        </div>   
        {{$scripts}}
      </body>
    </html>

See the empty `<ul>` tag? I'm going to fill that in later with the results from the order.in API.

Now I need to create a javascript file that will send a request to ordr.in when I click submit. I'll create a `main.js` file with the following content:

**/app/scripts/main.js:**

    function getRestaurantDetails(data) {
      _.each(data, function(restaurant) {
        if (restaurant.is_delivering) {
          var html = "<li>";
          html += restaurant.na + ": ";
          html += restaurant.cs_phone;
          html += "</li>";
          $("ul").append(html);
        }
      });
    }

    function getRestaurants(ev) {
      ev.preventDefault();
      var url = "/backlift/tunnel/restaurants/dl/ASAP/";
      url += $("input[name='zip']").val() + "/";
      url += $("input[name='city']").val() + "/";
      url += $("input[name='address']").val();
      $.get(url, getRestaurantDetails);
    }

    $(function (){
      $("form").bind("submit", getRestaurants);
    });

In the above file, I define a `getRestaurants` function that will be called when the form is submitted. The function will convert the form's data into a `get` request using the URL syntax for the ordr.in [restaurant API](https://hackfood.ordr.in/docs/restaurant). However instead of sending the request directly to ordr.in, I'm sending it to `/backlift/tunnel/restaurants` instead-- a tunnel URL that I'll configure in a minute. If the request succeeds, the result is passed to the `getRestaurantDetails` function. This function iterates through each restaurant and appends its name and phone number to the `<ul>` tag that I put in my `index.html` file.

Now I just need to define the restaurants tunnel. In my `config.yml` file I'll incude the following snippet:

    tunnels:
      restaurants:
        base_url: 'https://r-test.ordr.in'
        headers:
          X-NAAMA-CLIENT-AUTHENTICATION: 'id="XYZ123", version="1"'

This will route the request to the ordr.in server. To authenticate requests, ordr.in accepts a custom header including an API key. By using tunnels I can keep this stashed away in my config.yml file, rather than putting keys in my publicly accessible javascript code. (Note that you will need to replace 'XYZ123' above with your own API key)

Now when I fill out the address form and click "submit" I get a list of nearby restaurants that deliver to me. It looks like this:

<img src="/images/restaurantlist.png">

Sweet!

> **Why not just send the request directly to ordr.in?** If I tried to do that I'd get an error. This is due to the pesky same-origin policy. Actually that policy is pretty great because it greatly reduces vulnerability to cross origin attacks. 

## A "pizza finder" pivot

Looking through the results returned from my query to ordr.in, I can see a few obstacles to my "pizza finder" app. First off, I don't see any pizza restaurants in the results! Instead, I'm getting primarily asian food restaurants. Additionally the phone numbers are all the same, and they all start with 646 which is a New York area code. Perhaps I over estimated the demand for pizza in San Francisco, or maybe the world just isn't ready for a pizza finder app yet. No worries, I've got a head start now, so I'll be ready as soon as pizza joints start popping up in the ordr.in API. In the mean time I'm getting pretty hungry, so perhaps I'll pivot to the leftover indian food in the fridge.

## Conclusion

This quick demo shows how you can use the Backlift Tunnel API to access a third party API from your javascript. Right now the tunnel API works with JSON based APIs, and supports authentication via custom headers, basic auth, or special parameters. We're working on oauth support as well. Again, for more details, please check out our Tunnel API docs [here](http://backlift.github.io/docs/tunnel.html).
