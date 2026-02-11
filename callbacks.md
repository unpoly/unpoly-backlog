- Use onAccepted with JS string?
  - At least support X-Up-Open-Layer with callbacks
  - Or support actions, effects
    - What would be the difference between emitting an event with argument and invoking an action?
      => This is about setting an action when opening a layer from the server, with X-Up-Open-Layer
    - I feel we would duplicate several API methods with "actions" (reload, validate, emit, submit?)
  - Alternative with Render Intents:
    X-Up-Open-Layer: { intent: 'reload' }

    X-Up-Open-Layer: { intent: { reloadOnDismissed: { target: "foo" } } }

    up.fragment.config.intents.render
    up.fragment.config.intents.navigate
    up.fragment.config.intents.reloadOnDismissed

    => OK, das vereint render Options und das callback ding, aber nur die Callbacks wäre ja einfacher und eine Schachtelung weniger:

      X-Up-Open-Layer: { onDismissed: { 'reload' } }
      X-Up-Open-Layer: { onDismissed: { reload: { target: 'foo' } }

      up.action('reload', { target } => {
        up.reload(target)
      })
      
      - Also Actions sometimes need to be able to access the event:
      - 'up-on-accepted': up.safe_callback('up.navigate({ response: event.response })')

      up.callback('reload', (event, { target }) => {
        // It's weird because I sometimes want a { response } key and sometimes not
        
      })
    => The API with up.action/callback is *NOT* nicer than this. There's no extra indirection, and I don't need to know which param is part of the event and which is defined by the action
    
      up.layer.open({ onDismissed: up.safe_callback("up.reload({ response })") })

