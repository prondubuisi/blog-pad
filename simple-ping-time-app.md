
# Build Simple Ping Time App With Livewire Polling
![undraw_Time_management_re_tk5w](https://user-images.githubusercontent.com/22311928/173588230-da0ba3fc-4a02-4661-ad1f-812bbff3de4f.png)

## Problem

You are working on your next big side project, and there are several hosting alternatives. Speed is an essential factor for your application. Which hosting option has the best speed?

Your approach to determining the speed of your application is measuring the ping time interval. The ping time interval measures from when your client(browser) sends a request to your server to when your server returns a response to the client. How can you measure the ping time interval in a Livewire application?

## Solution

Our Ping time app measures the time interval of a request round trip. To calculate the request time interval, we will: 

* Make a client request to the server, keeping track of the request initiation time.

* Return a "Hello" response to the client once the server receives and processes the request. 

* Subtract the request initiation time from the current time in the client.

* Display the time difference as our Ping time. 


We need a component and a view to build our Ping time application. Laravel Livewire allows us to create both with one command.

`php artisan make:livewire PingTime`

This command creates two files, `PingTime.php`, and `ping-time.blade.php`. 


`PingTime.php`
```

<?php

namespace App\Http\Livewire;

use Livewire\Component;

class PingTime extends Component
{
    public function render()
    {
        return view('livewire.ping-time');
    }
}

```

`ping-time.blade.php`

```
<div>
    {{-- Be like water. --}}
</div>

```

The `PingTime.php` component class allows us to pass and manipulate data to and from the `ping-time.blade.php` view template.

Ping times vary depending on factors within our application and infrastructure. Running our Ping time application many times gives us a better estimation of the ping time interval compared to running the application just once. With Livewire polling, we don't need to run the application many times manually. 

Let's add a `div` with a polling directive to see this for ourselves.


```
<div>
    <div wire:poll.1s="">
    </div>
</div>
```

The `div` with the polling directive sends requests to our server every 1 second. This interval can be adjusted as needed. We can inspect our console to see the server requests.

![server call lightmode](https://user-images.githubusercontent.com/22311928/173617550-015cba96-09ba-4aa4-882e-8560dbb693ca.gif)


### Measuring Ping Time

We are looking to measure the ping time entirely on the client-side, so we need to track when our polling `div` initiates a request. Livewire polling allows us to specify an action to fire on the polling interval by passing a value to `wire:poll`; this should do the trick.

```
<div>
    <div wire:poll.1s="processClientRequest({{floor(microtime(true) * 1000)}})">
    </div>
</div>

```

`processClientRequest({{floor(microtime(true) * 1000)}})` passes the client request initiialization time in milliseconds to the `processClientRequest` method within the component class. Let's create that method. 

```
class PingTime extends Component
{
    public $clientRequestTime;

    public function processClientRequest($clientRequestTime)
    {
        $this->clientRequestTime =  $clientRequestTime;
        return "Hello";
    }

    public function render()
    {
        return view('livewire.ping-time');
    }
}
```

Declaring the `$clientRequestTime` property `public` makes it available to the component view. Since our request time is available to the view after the round trip, we are very close to meeting our Ping time app objective.


Let's display the difference between the client request time `$clientRequestTime` and when the server returns to the client. 

```
<div>
    <div wire:poll.1s="processClientRequest({{floor(microtime(true) * 1000)}})">
        <h1>Ping Time: {{floor(microtime(true) * 1000) - $clientRequestTime }} Milliseconds</h1>
    </div>
</div>
```

`{{floor(microtime(true) * 1000) - $clientRequestTime }}` substracts the client request initialization time from the current client time at each interval. Both times are in milliseconds.

![ping time gif](https://user-images.githubusercontent.com/22311928/173629804-41bfc936-d1f0-49f6-a5cd-d47a7a632605.gif)

### Ping App Improvements

We want the polling `div` to display and make server calls at the click of a button. Let's implement display toggling using the Livewire `$toggle` magic action and an `if` block. We also add a button to trigger the action.

```
<div>
    @if($showDiv)
        <div wire:poll.1s="processClientRequest({{floor(microtime(true) * 1000)}})">
            <h1>Ping Time: {{floor(microtime(true) * 1000) - $clientRequestTime }} Milliseconds</h1>
        </div>
    @endif
    <button type="button" wire:click="$toggle('showDiv')">
               Toggle Ping
    </button>
</div>

```

We need to set an initial  `$showDiv` property value in our component class for the `$toggle` magic action to take effect.

```
class PingTime extends Component
{
    public $clientRequestTime;
    public $showDiv = false;
    
    public function processClientRequest($clientRequestTime)
    {
        $this->clientRequestTime =  $clientRequestTime;
        return "Hello";
    }

    public function render()
    {
        return view('livewire.ping-time');
    }
}
```

![Poll on click](https://user-images.githubusercontent.com/22311928/173636760-30d7bdf9-c81a-4170-a503-efc6dba9e21d.gif) 

On click, the toggle button toggles the value of `$showDiv` between `true` and `false`; this controls the visibility of the polling `div`.  It is worthy of note that once our `div` is hidden, it does not poll in the background.   


The ping app displays an initial time difference that is quite large. This is because on page load  `$clientRequestTime = 0`. Our expression `floor(microtime(true) * 1000)` for getting the current client time in milliseconds counts from 0:00:00 January 1,1970 GMT, hence the large time difference. 

Let's solve this by ensuring we only get the time difference when `$clientRequestTime` is not equal to 0(zero). `$clientRequestTime` is only 0(zero) when the request has not completed a single round trip.

```
<div>
    @if($showDiv)
        <div wire:poll.1s="processClientRequest({{floor(microtime(true) * 1000)}})">
            @if($clientRequestTime)
                <h1>Ping Time: {{floor(microtime(true) * 1000) - $clientRequestTime }} Milliseconds</h1>
            @endif
        </div>
    @endif
    <button type="button"  wire:click="$toggle('showDiv')">
               Toggle Ping
    </button>
</div>

```

![fixed ping](https://user-images.githubusercontent.com/22311928/173647182-6fe2f0cf-eb4b-4c8a-8df8-fd2dc72ff2c6.gif)

Yay! We built a working Ping time app. 


## Discussion

Livewire builds on the Laravel developer productivity mantra to bring abstractions that make frontend development easy for Web artisans. The Ping time app looked initially complex, but Livewire features like polling and magic actions made our solution simple. 

