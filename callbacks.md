2026-02-12
---------

Can we have just one setting for all kinds of callbacks?

up.fragment.runScripts
up.fragment.runCallbacks


up.script.config.allow.default
up.script.config.allow.bodyScripts
up.script.config.allow.attributeCallbacks
up.script.config.allow.headerCallbacks



up.fragment.runBodyScripts
up.fragment.runCallbackStrings


up.script.config.allow.default
up.script.config.allow.bodyScripts
up.script.config.allow.callbackStrings







What I know (2026-02-11)
------------------------

- With a strict-dynamic CSP, we need to also enforce nonces for attribute-parsed callbacks.
- runScripts and the new allow.* configs conflate "Do I want to run scripts" and "What is allowed"
- I might need to check the nonce prefix before executing a <script>
  - Otherwise an attacker would just include an [up-on-loaded] with a random nonce
  - This means that we *need* the csp-nonce meta or we cannot execute
- What about onclick handlers? => No
- I'm not sure if allowing header-based scripting is a good idea.
  => It's probably OK.
  => Attackers that can tamper with headers could also disable CSP, set cookies, etc.










Original Notes
--------------


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

