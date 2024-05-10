---
marp: true
style: |

  section h1 {
    color: #6042BC;
  }

  section code {
    background-color: #e0e0ff;
  }

  footer {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 100px;
  }

  footer img {
    position: absolute;
    width: 120px;
    right: 20px;
    top: 0;

  }
  section #title-slide-logo {
    margin-left: -60px;
  }
---

# LiveState: LiveView Re-Imagined
Chris Nelson
@superchris
![h:200](full-color.png#title-slide-logo)

---

<!-- footer: ![](full-color.png) -->
# Who am I?
- Long-time (old) Elixirist
- Co-Founder of Launch Scout
- Creator LiveState (and friends)
- Apprenticeship enthusiast 

---

## LiveState started with a question: 
### How could I use LiveView in a app not served (or rendered) by Elixir?

---

# Why would you even want that?
- Embedded apps
  - Apps that live in your customers websites
- Static site generators
- Existing JS framework 
- It's about expanding the reach of Elixir

---

# What do I *really* love about LiveView
- The most important thing is not the template language
- It's how simple it makes the front end

---

## LiveView front end code only does two things
- Renders state
- Sends events

---

# What if we could use this pattern in more places?
## Could we keep our state in Elixir and our front-end dead simple?

---

# LiveState
- A hex package containing a channel behaviour
- A client side npm
- mostly a wrapper around Phoenix Channels
- sends events and receives state updates

---

# `live_state`
- `use LiveState.Channel` behaviour in your channel
- implement callbacks
  - init/3
  - handle_event/3

---

# `phx-live-state`
- javascript (typescript) npm
- increasing levels of abstraction:
  - `LiveState` - lower level API
  - `connectElement()` allows you to "wire up" a Custom Element
  - `@liveState` TS decorator lets you declaratively annotate a Custom Element class

---

# Example time: An [airport map](airport_map.html) element
- Custom element we can drop on any HTML page
- Displays pins for aiport data on a map
- Re-fetches when the user pans/zooms

---

## The HTML
```html
<html>

<head>
  <title>Student wobsite</title>
  <script type="module" src="http://localhost:4001/assets/custom_elements.js">
  </script>
</head>

<body>
  <h1>Wut up Gig City!</h1>
  <airport-map url="ws://localhost:4001/socket" api-key="AIzaSyDBDmRjILjZvT1tBpos4Y9LvnoU_wFTFe8"></airport-map>
</body>

</html>
```

---
## Airport map element
```ts
@customElement('airport-map')
@liveState({
  topic: 'airport_map',
  events: {
    send: ['tilesloaded', 'bounds_changed']
  }
})
export class AirportMapElement extends LitElement {
  
  @property({attribute: 'api-key'})
  apiKey: string = '';

  @liveStateConfig('url')
  @property()
  url: string = '';

  @state()
  @liveStateProperty()
  airports: Array<Airport> = [];

  render() {
    return html`
      <div class="map">
        <lit-google-map api-key=${this.apiKey} version="3.46">
          ${this.airports?.map(({name, identifier, latitude, longitude}) => html`
            <lit-google-map-marker slot="markers" latitude=${latitude} longitude=${longitude}>
              <p>${name} (${identifier})</p>
            </lit-google-map-marker>        
          `)}
        </lit-google-map>
      </div>    
    `;
  }
}

```

---
# Airports channel
```elixir
defmodule LiveElementsLabsWeb.AirportMapChannel do
  use LiveState.Channel, web_module: LiveElementsLabsWeb

  alias LiveElementsLabs.Airports

  @impl true
  def init(_params, _session, _socket) do
    {:ok, %{api_key: System.get_env("API_KEY")} }
  end

  @impl true
  def handle_event("bounds_changed", event_payload, state), do: load_airports(event_payload, state)

  @impl true
  def handle_event("tilesloaded", event_payload, state), do: load_airports(event_payload, state)

  defp load_airports(%{"north" => north, "east" => east, "west" => west, "south" => south}, state) do
    airports =
      Airports.list_airports_in_bounds(%{north: north, east: east, west: west, south: south})

    {:noreply, state |> Map.put(:airports, airports)}
  end
end
```
---

# So I created LiveState, looked upon it and it was good.

---

# But...
- Could I make developing my front end even simpler?
- Could I use LiveState without even having to create a Custom Element?
- What if you could just start from just an HTML file?

---

# Introducing LiveTemplates
- `<live-template>` connects a client side template to a LiveState channel
- Uses a client side templating library called [Sprae](https://github.com/dy/sprae)
- sends events
  - form events
  - clicks
  - more soon :)
- re-renders on state change
- optionally shares context

---

# What would a [CRUD app](silly_crm.html) look like?
- Created a people context using generator
- Person
  - first name
  - last name
  - birth date

---

# Let's see the code!
- silly_crm.html
- people_channel.ex

---

# One more thing

---

# Signals: a standard way to do reactivity
- Values that track access and trigger renders on change
- Rapidly being adopted across JS frameworks
- SolidJS, Preact, Angular, etc
- Current implementations differ
- TC39 Proposal

---

# LiveSignals
- bridging a signal to the backend
- Supports Preact and TC39 so far 
- Supporting something else would be a very tiny PR :wink:

---

# Let's see a [Preact demo](preact-example.html)

---

# Thanks!

- slides: https://github.com/superchris/gce-2024-livestate
- live_state elixir library: https://github.com/launchscout/live_state
- phx-live-state client npm: https://github.com/launchscout/live-state
- live-templates: https://github.com/launchscout/live-templates
- live-signals: https://github.com/launchscout/live-signals

---

# Bonus time!

---
