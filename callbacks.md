Was ist Policy?

- Ein Wert: 'auto' | 'pass' | 'block' | 'nonce'
- Ein Weg, 'auto' zu resolven
- Evtl. eine Warning nach auto-Resolution
- Eine Log-Msg bei blocken (z.B. "Blocking CALLBACK via up.script.config.PROPERTY='auto': SOURCE")





2026-02-17 (2)
=========

Some more thoughts so I can sleep:

- It's simpler to capture the intent for callbacks than for scripts.
- Hence every attempt to unify to a common 'auto' behavior will feel weird for one of the two.
- Auto-blocking on detection of 'strict-dynamic' is a perfect mitigation against `<script>`
  - However, when we couple this to 'auto' only, users that opt into 'pass' cannot use it
  - Is this important?
- The detection of 'unsafe-eval' is not as clear. The user might just have a completely open CSP.
- Users with a strict hostname-based CSP may be surprised when they add a `<meta name="csp-nonce">` for callbacks, but then their scripts no longer load

So the following idea:

- We offer 'auto' for both.
- Auto only flips between "nonce" and "pass" by detecting the presence of `<meta name="csp-nonce">` (and possibly document.currentScript?.nonce)
- Scripts default to 'block'. This is compatible with 3.11. Users can manually opt for "nonce" (strict-dynamic), "pass" (host-based) or "auto" (maybe for examples).
- Callbacks default to 'auto' (was: 'pass'). This should be also compatible with most users.
- Unsafe CSPs ('strict-dynamic', 'unsafe-eval') only log a warning, but don't change blocking
- There is no policy.default

How would we name these two settings? Some ideas:

- up.script.config.scriptElementPolicy
- up.script.config.evalCallbackPolicy

This change is safe and backwards compatible.
It's a little sad that we actually had a full mitigation for scripts (allow, but block for strict-dynamic), but we decide to not use it anymore.

So maybe think about defaulting to scriptPolicy: 'auto', but it works like this:

- Ignore <meta name="csp-nonce">
- Allow by default, assuming either a host-based CSP or no CSP
- Require a 'nonce' when we see strict-dynamic

Also:

- Still print the warnings when we're in 'pass', but see potentially unsafe directives
- Include install instructions
- Add /script-security




2026-02-17 (1)
=========

A new approach that captures the intent of everybody: We have two different meanings for 'auto'.


Default
-------

up.script.config.policy.default = 'auto'

  Used for both callbacks and scripts, but means something different for each.


Callbacks
---------

up.script.config.policy.stringCallbacks = 'auto' (default inherited from policy.default)

  If a <meta name="csp-nonce"> is given: Require this nonce for callbacks (behave like 'nonce')
  If no <meta> is given: Pass to CSP (dangerous for unsafe-eval) (behave like 'pass')
  Maybe ignore document.currentScript.nonce, since that might just be for loading Unpoly from CDN?
  
If we have stringCallbacks = 'pass' (or when auto resolves to 'pass') and we observe 'unsafe-eval' in a response, we print:

  [up.script] An 'unsafe-eval' CSP allows arbitrary [up-on...] callbacks. Consider setting up.script.config.policy.stringCallbacks = 'nonce'.
  However, we do not change our behavior. It's just the warning.
  This might annoy people with a "don't care" CSP, but those could set up.script.config.cspWarnings = false
  

Scripts
-------

up.script.config.policy.bodyScripts = 'auto' (default inherited from policy.default)

Backwards compatible behavior (like 3.11, but allow nonces):

  Fully behave like 'nonce' ? This is actually slightly LESS strict than our new default in 3.11

  Ignore <meta name="csp-nonce"> or document.currentScript.nonce.
    This might just be given to run callbacks in an otherwise host-based strict CSP. **Right?**
    Yes, but if we start out with "run no scripts" from 3.11 that's still an improvement
  
Alternative (analogue to callback auto):

  If a <meta name="csp-nonce"> is given: Require this nonce for callbacks (behave like 'nonce') -- Might accidentally disable scripts for a host-based strict CSP. **But those are default-disabled anyway in 3.11+**
  If no <meta> is given: Pass to CSP (dangerous for strict-dynamic) (behave like 'pass') -- This allows more than 3.11 for strict-dynamic, but not if we still clean on strict-dynamic

If we have bodyScripts = 'pass' (or when auto resolves to 'pass') and we observe 'strict-dynamic' in a response, we print:

  [up.script] A 'strict-dynamic' CSP allows arbitrary `<script>` elements in new fragments. Consider setting up.script.config.policy.bodyScripts = 'nonce'.
  However, we do not change our behavior (and anyway we couldn't cover callbacks outside the fragment or before we swap). It's just the warning.
  

Alternative to warnings
-----------------------

A different meaning of 'auto' could also be:

- If we observe a dangerous CSP directive, we clean just in case
- This is still a full mitigation for scripts (which run immediately and then never again)
- This is only a partial mitigation for callbacks

We don't do this for "pass".


  
Migration
----------

up.fragment.config.runScripts = true
  => Warn that you should use up.script.config.policy.bodyScripts = 'pass' instead, but the safer alternative is 'auto' or 'nonce'
  => up.script.config.policy.bodyScripts = 'pass' (or 'auto' if we choose the liberal default) (same minus the strict-dynamic safeguard)
  => up.script.config.policy.callbackStrings = 'auto' (new setting, possibly just rely on the policy.default inheritance)
  
up.fragment.config.runScripts = false
  => Warn that you should use up.script.config.policy.bodyScripts = 'block' instead.
  => up.script.config.policy.bodyScripts = 'block' (same)
  => up.script.config.policy.callbackStrings = 'auto' (new setting, possibly just rely on the policy.default inheritance)
  
Generally ask people on :boot to configure up.script.config.policy if they never configured anything there, and neither runScripts.



2026-02-16 (2)
===========

Dangerous combinations:

- unsafe-eval allows execution of *all* string callbacks (just keep out the nonce)
- strict-dynamic allows execution of *all* scripts (and string callbacks with incorrect nonce unless we check it, which we now do)

We also cannot detect both these for the initial page load.

How about this:

We only have two settings:

    up.script.config.policy.bodyScripts = 'pass' (default)
    up.script.config.policy.stringCallbacks = 'pass' (default)

But when we encounter 'unsafe-eval' in a ResponseDoc we print a warning:

    [up.script] An 'unsafe-eval' CSP allows arbitrary [up-on...] callbacks. Consider setting up.script.config.policy.stringCallbacks = 'nonce'.
    [up.script] A 'strict-dynamic' CSP allows arbitrary `<script>` elements in new fragments. Consider setting up.script.config.policy.bodyScripts = 'nonce'.

The warnings are only printed once.
The warnings are not shown if user configured a "nonce" or "block" policy.
The warnings are shown whenever we encounter a problematic directive. We don't wait for an actual `<script>` or `[up-on-*]` callback, as these might be inserted by an attacker.

Open questions:

- Do we still want the "auto" option?
  - It does not work for [up-on] callback in the initial page load, or in attacker-controlled HTML outside the updated fragment
  - It might give a wrong sense of security
    - But we still print the warning
    - At least we protect the new fragment.
    => Should we switch to "nonce"? Or is this too much magic?
- Install: Security Guide
- up.script.config.unsafeCSPWarnings

Ideen:

- Default block
  - 1st experience bad
- Automagic reconfigure


2026-02-16 (1)
=============

JC
  strict-dynamic + unsafe-eval
    Means initial attrs unprotected
    
Many projects
  define a nonce "just in case"
  or for one-off scripts
    => but that doesn't require a `<meta name="csp-nonce">`
  or for unpoly callbacks
  requiring a nonce by default would now break host whiteist, like stripe.com



2026-02-12
===========

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
========================

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
==============

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

