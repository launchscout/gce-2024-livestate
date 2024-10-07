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

# Extending the reach of Elixir with LiveState
Chris Nelson
@superchris
![h:200](full-color.png#title-slide-logo)

---

<!-- footer: ![](full-color.png) -->
# Who am I?
- Long-time Elixirist
- Co-Founder of Launch Scout
- Creator of LiveState (and friends)
- Apprenticeship enthusiast 

---

# LiveState
## Letting you *use* Elixir when your app isn't served *by* Elixir

---

# Why would you need that?
- Embedded apps
  - Apps that live in your customers websites (Wordpress, Wix, etc)
- A *much* better DX than custom javascript, iframes, and APIs
- **It's about expanding the reach of Elixir**

---

# But first: What do I *really* love about LiveView
- The most important thing is not the template language
- It's how simple it makes the front end

---

# How does LiveView keep things simple?
- State lives on the server
- LiveView code:
  - Renders state
  - Computes new state from events

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
  - handle_message/3

---

# `phx-live-state`
- javascript (typescript) npm
- increasing levels of abstraction:
  - `LiveState` - lower level API
  - `connectElement()` allows you to "wire up" a Custom Element
  - `@liveState` TS decorator lets you declaratively annotate a Custom Element class

---

# More about `phx-live-state`
- [Custom Events](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent) are pushed to the channel
- state changes are send as JSON patch
  - with a version to prevent out of order messaging issues
- elements can share a LiveState instance via the Context Protocol
  - `provide` and `context` attributes of config
  
---

# Example time: `<chat-room>`
- The classic Phoenix example
- now packages as a custom element (aka Web Component)

---

# An aside on Custom Elements
- Great way to add your app to your clients websites
- Worpress, Wix, Squarespace, Drupal
- Turns out they all support HTML!
- And you can style it with good ole CSS!

---

# The chat channel
```elixir
defmodule ChatRoomWeb.ChatRoomChannel do
  @moduledoc false

  use LiveState.Channel, web_module: ChatRoomWeb
  alias Phoenix.PubSub

  @impl true
  def init("chat_room:" <> room, _params, _socket) do
    PubSub.subscribe(ChatRoom.PubSub, "chat_messages:#{room}")
    {:ok, %{messages: [], room: room}}
  end

  @impl true
  def handle_event("send_message", message, %{room: room} = state) do
    PubSub.broadcast!(ChatRoom.PubSub, "chat_messages:#{room}", message)
    {:noreply, state}
  end

  def handle_message(message, %{messages: messages} = state) do
    {:noreply, state |> Map.put(:messages, [message | messages])}
  end

end
```
---
# `<chat-room>`
```js
export class ChatRoomElement extends LitElement {

  static properties = {
    url: {},
    room: {},
    messages: {
      type: Array,
      attribute: false
    }
  }

  connectedCallback() {
    super.connectedCallback();
    connectElement(this, {
      url: this.getAttribute('url'),
      topic: `chat_room:${this.getAttribute('room')}`,
      properties: ['messages'],
      events: {
        send: ['send_message']
      }
    })
  }
```

---
# `<chat-room>` cont.
```js
  get messageElement()  {
    return this.shadowRoot.querySelector('textarea[name="message"]');
  }

  render() {
    return html`
    <ul part="messsage-list">
      ${this.messages?.map(({ author, message }) => html`<li>${author}: ${message}`)}
    </ul>
    <form @submit=${this.sendMessage}>
      <div>
        <label>Author</label>
        <input name="author" />
      </div>
      <div>
        <label>Message</label>
        <textarea name="message"></textarea>
      </div>
      <button>Send!</button>
    </form>
    `;
  }

  sendMessage(e) {
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData.entries());
    e.preventDefault();
    this.messageElement.value = '';
    this.dispatchEvent(new CustomEvent('send_message', {detail: data}));
  }
}

window.customElements.define('chat-room', ChatRoomElement);
```

---

# [`chat-room.html`](chat_room.html)
```html
<html>
  <head>
    <script type="importmap">
      ...
    </script>
     <script type="module">
      import './chat-room.js'
    </script>
  </head>
  <body>
    <h1>Wut up Denver Elixir!</h1>
    <chat-room url="ws://localhost:4000/live_state" room="denver_elixir"></chat-room>
  </body>
</html>
```

---

# So that's cool, but...
- What if I don't need/want to make a custom element?
- What if I could just connect the html on my web page to a LiveState channel?

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

# Kinda like htmx meets Elixir

---

# Chat with [live-templates](live_template.html)
### In which I live code...

---

# Ok but what if like Preact/Solid/Svelte/XXX..

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

# And now for something completely different...
## Let's turn our `<chat-room>` into a WordPress plugin

---

# Built with LiveState
- [LiveRoom](https://liveroom.app)
- [Launch Elements](https://elements.launchscout.com)
- [PatientReach 360](https://patientreach360.com)

---

# Thanks!

- slides: https://github.com/superchris/gce-2024-livestate
- live_state elixir library: https://github.com/launchscout/live_state
- phx-live-state client npm: https://github.com/launchscout/live-state
- live-templates: https://github.com/launchscout/live-templates
- live-signals: https://github.com/launchscout/live-signals
- [Wordpress blog post](https://launchscout.com/blog/elixir-wordpress-plugin)

---
