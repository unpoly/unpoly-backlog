Priority
========

- [up-disable-for] [up-enable-for]
  - https://github.com/unpoly/unpoly/discussions/682

[ok] - Support X-Up-Open-Layer: { type, ...VISUAL_OPTIONS }
  [ok] - Support unpoly-rails (up.layer.open())
  - Document that the target resets to :main

[ok] - Layer options for [up-layer=new] talk about position only for popups, but it also affects drawer

[ok] - Support up.migrate.config.logLevel = 'none'

- Copy important options from up.render() into up.validate(), up.submit(), up.follow()
- Copy important attributes from [up-follow] into [up-submit]

- Revert the autoFail change
  - We don't do this for other global defaults, like expireCache, evictCache
  
- X-Up-Origin-Mode ?

- In up:request:load, explain that request headers may still be changed

- Default "[options.layer='origin current']" should use commas, no?

- Go through all feature visibilities... up.fragment.* functions in particular

- Implement up:form:switch event by refactoring the way switcher/switchee lookup works

- Consider a way to "panic" out with a developer error
  - Replace entire <html>
  - Accept that JS stays loaded
  - Reset Unpoly so links aren't followed
  
- Do we support a { focusVisible } option that is not documented?

- Can we document options.fail* ?
  

Docs rework
===========

- Consider making the docs full-width
  - Also replace the breadcrumb with a link that opens the drawer
  - Move the entire search to Algolia



Backlog
=======

- Consider hiding the focus ring for main elements by default

- Consider validations to not expire the cache (do they even?)

- Test that we don't process { abort } when the RenderPass was aborted by { guardEvent } or { confirm }

- I'm not sure if opening a layer with local content honors { abort }
  - E.g. start loading content on the base layer's main
  - Open a layer from a string (which makes a request with bindFragments: [base-layer's main])
  - Content on base layer is still loading

- Think about a way to optimistically render an action that dismisses an overlay and changes something in the background
  - What about dismiss callbacks?

- Offer a way to expand a template into an element
  - Like <up-defer class=".container-from-template" up-fragment="#template">, but without the need to have a good selector
  - Can we somehow say that we want to replace ourselves, even when the replacement has a different selector?
    - HTML-Importe waren   <link rel="import" href="http://example.com/elements.html">
  - There is no unobtrusive way to do this

- Remove commented-out code in up.util

- affix() and createSelector() cannot create an attribute called [content] or [text]
  - e.g. for meta[content]
  - Maybe force all attrs into { attrs } ?
    - Or allow { attrs }

- Rework compiler docs (unpoly/unpoly#688)
  - Extract docs from up.compiler() into its own guide page /compilers "Initializing JavaScript" (with compilers)
  - Fix existing links to /up.compiler and /up.compiler#destructor
  
  - Make examples for all three destructor forms:
    - Returning a function
    - Returning an array of functions
    - Using up.destructor
  - Section for async compilers
    - Timing issues if the element is detached before it is compiled
    - If they return a function, it will not run as a destructor
    - Maybe asyncCompiler helper
    - Note that this will be improved in the future

    - ```
      // Registers an `up.compiler()` function that returns a promise for its destructor function.
      export default function asyncCompiler(...compilerArgs) {
        const compilerFn = compilerArgs.pop()

        up.compiler(...compilerArgs, (...passArgs) => {
          // Call the compiler function passed by the user.
          const returnValue = compilerFn(...passArgs)

          // Return a new destructor function that awaits the asynchronous
          // return value and uses the fulfillment value as a destructor.
          return async () => {
            // The async function could return different types of values:
            //
            // (1) A destructor function
            // (2) An array of destructor functions
            // (3) A value that is not a function (which we discard)
            let destructorCandidates = await returnValue
            destructorCandidates = up.util.wrapList(destructorCandidates)

            for (const destructorCandidate of destructorCandidates) {
              if (up.util.isFunction(destructorCandidate)) {
                destructorCandidate()
              }
            }
          }
        })
      }
      ```    

- Allow to keep unpoly-migrate without logging, e.g. up.migrate.config.logLevel = 'none'

- Improve active classes, loading classes
  - Tailwind users would love [up-active-class] [up-loading-class]
  - Maybe allow a function for config.activeClasses, config.loadingClasses

- Make a doc page for custom elements (unpoly/unpoly#688)
  - Explain that you can just use defineElement
  - Explain that you can use compilers for simple cases
  - Show how custom elements can participate in form submissions
    - config.fieldSelectors
    - Support { name, value, disabled }
    - Note that Unpoly does not support formAssociated or formdata event, but that this will be improved in the future
    
- Make a doc page for polling
  - Extract from [up-poll]
  - Own section for "Reloading when re-focusing a tab"?

- I somehow expected up.fragment.config.runScripts to be in up.script.config.runScripts
  - We would need a new name like up.script.config.runFragmentScripts
  
- Add Rust install to https://github.com/unpoly/unpoly-site/tree/master/source/install

- Consider a callback to manipulate render options before interaction
  - Either element-specific ([up-on-follow], [up-on-submit], etc.)
  - Or generic ([up-on-use="..."], [up-setup="..."], [up-before], [up-guard], [up-on-run] [up-on-activate="..."], [up-on-start])
    - Use is short, but conflicts with [up-use-data] and does not work for polling
  - Callback to run before any render pass (see davidsums popup discussion)
    - up:render:prepare or up:render:setup or up:render:options
    - Would we rename up:fragment events to get pressure off the up:fragment namespace?
      - up:render:load
      - up:render:loaded
      - up:render:keep       :(
      - up:render:offline
      - up:render:inserted   :(
      - up:render:hungry     :(
      - This would leave up:fragment:poll

- FM required overriding of history from the server
  - They really wanted { navigate }, no?
    - But can we really change navigation options at that point?

- Remove params parsing for up.watch(), support formdata, ElementInternals
  - We can just parse the entire form with new FormData(), then filter on contained elements
    - We will still need to know form fields for that, or filter on [name]
  - Replace up.Params.fromForm() with `new this(new FormData(form))`

- Consider [up-zone] as an origin-aware lookup without further logic
  - Also support `:zone` as a new pseudo
    - Layer priority beats zone priority
  - config.zoneSelectors = ['[up-zone]', ':main]
  - config.noZoneSelectors = ['[up-zone=false]']
  - I think without an { origin } we only want to check out `:main`. Maybe it's not configurable?
  - Do we want to allow [up-zone="name"] (and include it in targetDerivers), or should it be [up-zone][id="foo"]?
    - If we want to get rid of [up-href] (to enable morphing), we should encourage the use of IDs
  - If we want to mount arbitrary pages to mount in an [up-zone] it should be a parent.
    - e.g. <up-zone><main>content</main></up-zone>
      - But then we would have duplicate <main>
        - section[up-main=zone]?

- Maybe all guardEvents should have a loadPage() method
  - So it's more discoverable then event.loadPage()

- Consider publishing { dataMap, placeholderMap, previewMap } as @experimental
  - Check if we have tests here
  - Check whether a single { map } option (or an object target) would be superior
    - up.render({ target: { '.foo': { data: { key: 123 }, placeholder: 'spinner' }, '.bar': {} }, url: '/path' })
    - up.render({ target: { '.foo, .bar' }, dataMap: { '.foo'. { key: 123 } }, placeholderMap: { '.foo': 'spinner' }, url: '/path' })
    - up.render({ target: { '.foo, .bar' }, map: { '.foo'. { data: { key: 123 }, placeholder: '.spinner' } }, url: '/path' })
    => Doing something to the primary is a very common use case and great now

- Consider publishing up.script.clean()
  - Would we need a fallback for { layer }?
  - Supporting the use case "recompile later" is hard because then we would need to update the idempotent-compiler-fn-tracking
  
- Consider removing { content } option of createFromSelector(), affix(), etc. in favor of a third argument for constructing nested things
  - When we make this change we should also fix the inconsistency that a string arg is considered HTML, but Array<string> is considered text (and will be escaped)

- Allow to set custom close animation from link
  - Also support (or document?) up.layer.open({ closeAnimation })
  
- Consider contracting more attributes into shorthand values
  - E.g. [up-emit] and [up-emit-props] should be just one thing
  - E.g. [up-scroll] and [up-scroll-behavior] should just be one thing (but what about all the reveal attributes? reveal-snap/top/pading/max)
  - E.g. [up-transition] and [up-easing] and [up-duration] should just be one thing
  
- Evtl. sollte [up-back] den letzten History-Eintrag im Root (?)-Layer nehmen?
  - Back vs. Modals allgemein betrachten
  
- In the layer config, openAnimation and closeAnimation is a function, but it receives no useful args
  - Give it { layer, value }
  - Document it

- Consider up.RenderResult#primary or up.RenderResult#fallback
  - Updating ":main" via { fallback: true } must yield #primary === true
  - So maybe #primary, #expected, #intended or similiar is a better name than #fallback

- Rename "loading partial" to "filling partial" or similiar
  - up:partial:fill
  - up.partial.fill()
  - So "load" remains the term for sending a request
  - So we can offer [up-on-fill]
  
- Consider X-Up-Redirect so we can get rid of param-transporting in unpoly-rails
  - Unpoly would track all X-Up params of request and response

- Do we need a meta.previewing prop for compilers?
  - This would need to be a global prop because we allow arbitrary code
    - up.status.previewing
  - We could also pass it explicitly when { hello }ing from Preview functions
    - insert()
    - openLayer()
    - This would be less code than a global flag
- Exposing the temp functions would make it easier to compose some effects without accepting an (preview) arg
  - Expose up.form.disable() (since we also expose it via Preview#disable())
  => I'm not sure how useful this is in practice. Not needed in the demo.

- Can we find a way to implement the new-task action as a server-rendered template?
  template = preview.cloneTemplate('#foo', { '{{text}}': params.text })
  
    clone = preview.cloneTemplate('#foo', { '.task-item--text': params.text })
    clone = preview.cloneTemplate('#index', { 'tr': rows => rows.slice(20).forEach((row) => row.remove() })
    
    clone = preview.cloneTemplate('#foo')
    clone.querySelector('.task-item--text').innerText = params.text

    clone = preview.cloneTemplate('#index')
    [...clone.querySelectorAll('tr)].slice(max).forEach((row) => row.remove())


    // 169 bytes
    function cloneTemplate(template, processors = {}) {
      let result = template.content.cloneNode(true)
      for (let [selector, processor] of Object.entries(selectorMap)) {
        let matches = u.toArray(result.querySelectorAll(selector))
        if (u.isFunction(processor)) {
          processor(matches)
        } else {
          matches[0].innerHTML = processor
        }
      }
    }
- Consider publishing booleanOrString or booleanOrNumber
- Consider waiting for custom elements to be defined
  - For focus
  - For scrolling
  - For watching
- Support render lifecycle attributes for [up-defer] and [up-poll]
  - [up-on-loaded]
  - [up-on-rendered]
  - [up-on-finished]
  - [up-on-offline]
  - [up-on-error]
  - Right now this can be set by listening to a guardEvent and manipulating event.renderOptions
- We often see multiple preloading entries in the log (mouseover, mousedown). Can we not log if we're already preloading?
- Support array fields with watch()?
  - up.form.config.arrayFields = 'suffix' | 'all'
- Does it matter that guardEvents do not set up.layer.current?
  - It should mostly be the right layer anyway, except for polling in the background or something
- [up-validate] from a different URL
  - https://github.com/unpoly/unpoly/issues/486
  - This should make for a simpler code example
- An a[href][up-defer] should be *un*clickable?
  - Remove tabindex
  - Remove pointer cursor
- Remove isNull() and isUndefined()
  - The test is trivial, but it may be nice to not have to think aobut it
- Allow to keep elements *without* remembering to mark elements as [up-keep]
  - config.keepSelectors
  - Allow { useKeep: '[up-keep], .search-input' } to keep additional elements
  - Elements can still opt out with [up-keep=false]
- Docs: [up-transition] should document params [up-duration], [up-easing]. Same with [up-animation]?
- Docs: up.morph() should document options.duration, options.easing. Same with up.animate()?
- Allow up.reload() to reload multiple, potentially optional elements
  - Starting in 3.8 we could also use 3 times up.reload() and make a single request 
- Consider publishing { defaultMaybe: true } with a better name
- Docs: Extract new doc page "Preserving state" from [up-keep] and /data#preserving-data-through-reloads
- Docs: Installation methods may be unmaintained; Go through methods and focus on installing frontend assets
- Docs: URLs like /up-transition should redirect to /a-up-transition 
- Should up.layer.open({ data }) set the data on the root element, not the layer element?
  - Users can still "compile" the layer element by observing up:layer:open 
- Docs: Extract /polling page from [up-poll]
- Have a better error when Unpoly is loaded twice
  - We cannot do this when booting
  - Throw in namespace.js
- Add a config to disable validate merging
- Touch events can scroll the background of a drawer overlay on https://unpoly.com/
- In a multi-step render pass, let compilers see all updated fragments
  - This would require us to delay compilation until all fragments are inserted 
- Consider whether Request#target, Request#context etc. should be setters that auto-set the corresponding header.
  - Would save code in mergeIfUnsent()
  - Would save code in setAutoHeaders()
  - We could also make #headers or #header() hallucinate new headers
    - Maybe hard since we allow write access headers[key] = value
      - Would be easier with header() and setHeader()
      - Still need to iterate over the whole thing for passing over headers to xhr
- Consider an up:fragment:render event to modify renderOptions for all kind of passes
  - We have so many guardEvents now
  - Test that we can still modify event.renderOptions
  - Point against that name: We already have { onRendered } and that fires after DOM mutation, not before the request
  - Possibly offer originalEvent or something?
    - No! We would need to check their renderOptions for mutation. Who is interested in other events can use them.
  - Update docs: Render Flowchart
  - Update docs: Manipulate render options
- { dismissLabel } should be able to contain HTML
  - Maybe rename to { dismissButtonContent } or something?
- Allow variations of official layer modes
- lightbox Layer mode
  - Where are we going to put the margin around the box, if we're not having a viewport? 
- Move custom spec helpers out of `jasmine.foo()` to `specs.foo()`
- Unpoly Rails needs to ignore pseudo-elements
- I think we can replace up.Rect.fromElement() with just element.getBoundingClientRect() 
- I think revalidation now loses :maybe marks. We should have more tests.
  - Case 1 => I think this is implemented
    - We're loading ".foo, .bar:maybe"
    - Server only renders ".foo"
    - Revalidation should just be for ".foo", or for ".foo, .bar:maybe"
    - It should be OK if revalidation response only contains ".foo"
  - Case 3 => I could live without this.
    => Or should we allow reavalidation with defaultMaybe: true ?
    - We're loading ".foo, .bar:maybe"
    - Server renders ".foo, .bar"
    - Revalidation should just be for ".foo", or for ".foo, .bar:maybe"
    - It should be OK if revalidation response only contains ".foo"
  - Case 2 => I think this is implemented
    - We're loading ".foo, .bar:maybe"
    - Server renders ".foo, .bar"
    - While we're loading .bar was removed from the page
    - Revalidation should just be for ".foo", or for ".foo, .bar:maybe"
    - It should be OK if revalidation response only contains ".foo"
- Preserve text selection ranges when reloading / polling / revalidating
- Should [up-poll] and up.reload() render failed responses if they match the target?
  - At least for reloading the element may actually have entered the DOM from a failed response
- Site: Scrollbar styling https://makandracards.com/makandra/617528-chrome-121+-no-longer-supports-non-standard-scrollbar-styling
- Per-link animation options for new layers
  - Support mutatable animation options for up:layer:dismiss, up:layer:accept
  - Parse [up-close-animation] for links that open a new layer
  - Parse [up-open-animation] as an alias for [up-animation]
- The log message "Could not match primary target" should not appear when we're only working with fallback targets, and we happen to use the second one. E.g. first one is [up-main=modal], second one is [up-main].
- Print a warning when we can find no better form group than the <form> itself
- Run macros (but not compilers) *before* history changes
  - This is super hard due to the way UpdateLayer and OpenLayer works
  - In particular we also give the guarantee that the current layer renders before other (hungry) layers
- Give layers an [index] attribute so users don't style based on [nesting]
  - Do we give root an [index] as well?
    - We cannot use [index] here
    - [up-layer-index] would maybe a good default
      - Check vs. V4 plans
- Docs should show DOM structure of all layer modes
- Simplify animation API implementation with Element.animate()
  - Animation finishing could be implemented with Element#getAnimations()
  - This should also fix a regression for: Animations that fly in an element from the screen edge (move-from-top, move-from-left, etc.) no longer leave a transform style on the animated element. By @triskweline.
- Don't allow a request payload for DELETE requests
  - Maybe make this configuratable as opinions vary here.
    - e.g. up.network.config.payloadMethods
  - Is this really important?
- Consider parsing scroll-margin-top, scroll-margin-bottom for revealing
- Allow [up-emit] for buttons
- Docs: Explain how to convert JSON responses to HTML responses (discussion #562)
- Convert .sass files to .scss
- While morphing or destroying, disable pointer events on old element
- Support { onLoaded }, [up-on-loaded] for polling fragments
  - Test: Can be used to skip() a polling response, polling then continues
  - Test: Can be used to preventDefault() a polling response, polling then continues
- [up-disable] should disable links (discussion #561)
  - Within the form
  - As a stand-alone link
  - How to mark disabled links? .up-disabled ? Just [disabled]?
  - CSS: { cursor: not-allowed }
  - Also [aria-disabled=true]
  - Prevent with JS? pointer-events?
    - Also prevent keyboard focus with temporary [tabindex=-1]
  - Bootstrap already has a.disabled and users would like to take over that styling
- Docs: Copy important up.render() options to up.layer.open() and a[up-layer=new]
  - In particular { content, fragment, url, document }
- Look within :origin first
  - Remove similiar logic in up.FragmentProcessor: findSelector()
- Remove [up-validate] from individual fields https://unpoly.com/up-validate#validating-multiple-fields
- Support nonce rewriting for arbitrary attributes, e.g. up.safe_callback('124321321', '432423423')
  - But then we would need to know all the attributes we're rewriting to query them quickly
    - Or find them with XPath
  - OR we could support [up-on] with an event name and a nonce
- Swapped body looks funny in Safari (has .up-destroying class and is grayed out with old content in inspector)
- Allow to disable batching in validations
- inlineStyle, setInlineStyle should work with custom properties
- Jasmine matchers no longer get a `customEqualityTesters` argument
- up.ResponseDoc: Only re-discover the origin when match is 'region'
- Async compiler functions
  - See separate doc
  - Don't need to delay this until V4 as destructors no longer throw
- Is the [up-keep=false] pattern good to keep?
  - E.g. for flashes: We could interrupt a keep chain this way, but then the element can never again be keepable
  => It's the same for polling
- consider having dev builds with debug terser
- Docs: /accessibility page
  - But where does it go?
  - Top-level?
- 2024: Replace up.error.emitGlobal() with window.reportError()
- Publish up.util.sprintf()
  - But rename it to something else, as sprintf() is something else
- Docs: Custom input elements
  - Have your custom element expose `{ name }` and { value }` properties (AFAIK Shoelance components already do)
  - Also `{ disable }`
  - Make Unpoly aware of custom elements by pushing a selector to [`up.form.config.fieldSelectors`](https://unpoly.com/up.form.config#config.fieldSelectors).
  - Make Unpoly aware of custom submitt buttons by pushing a selector to [`up.form.config.submitButtonSelectors`](https://unpoly.com/up.form.config#config.submitButtonSelectors).
  - Limitation: One value per field. Checks for groups (.checked and .selected) are still hard-coded
- Test that system compilers always run before user compilers
  => We already have macros if compilers need to use others?
- Concepts [up-feedback] and [up-background] are overlapping
  - Should feedback: false imply background: true ?
  - Should we generalize the feedback option?
    - { feedback: ['classes', 'progress', 'custom'] }
      - But I always want classes?
- Message when unknown target: Revalidating cached response for target "undefined"
- Extend [up-switch] with [up-disable-for="..."] [up-enable-for="..."]
  - This is really confusing with [up-disable]
  - Or should we have an [up-switch-for] and [up-switch-effect]?
- up:feedback:start / up:feedback:stop
  - On the layer, not an individual element
  - Needs to have reference to origin and targetElements
    - Must be the same element in :start / :stop, even if the DOM has changed in between
  => With .up-validating we have different points where feedback starts and ends.
    - This class does not exist. It was an idea detailed further down.
  => Or we could have a single up:feedback event with a promise for the duration
- Do we want a preventable up:request:reuse event?
  - Maybe even :reused since there is no :loaded for cached requests
- Experiment with property mangling vs. public API
  - Possibly do a unpoly.experimental-min.js ?
  - If we could test a minified version we could publish this
    - Do we really think about this when publishing?
    - This would need to be part of the regular build
- Issue with splitting Immediate-Child-Selectors: https://makandra.slack.com/archives/C02KGPZDE/p1659097196752189
- Can we get rid of the afterMeasure() callback?
- Support { scroll: 'keep' }
  - Similiar to { focus: 'keep' }
  - Store scrollTops around a viewport in UpdateLayer
  - Possibly refactor tops to up.ScrollTopsCapsule
- Check that the target (and all non-successful target attempts) are visible in the log
  - Possibly highlight
- Do we want to support ` or ` in targets?
  - Is there any case other than [up-target], where we already have [up-fallback] ?
- Should animations force painting?
  - I don't understand how the transitions work when we set both the first and final frame in the same JavaScript task
- Maybe auto-submit should not navigate by default
  - How to restore this?
  - up-navigate ?
  - up-watch-navigate ?
  - up-autosubmit-navigate ?
  - Is it cool that search results would no longer get a URL update?
- Does [up-back] use the previous layer location?
- There was a case where { origin } should look inside an origin:
    up.render('.day_menu_dish_autocomplete .day_menu_dish_autocomplete--suggestions', {
      origin: titleInput,
      url: '/day_menus/suggestions'
    })
- Expose up.fragment.config.renderOptions
  - Now that we have *some* opinion about the defaults { abort: 'target', focus: 'keep', revalidate: 'auto' }, users may want to deactivate
- Try to have a 'hash' strategy in overlays
  - Also update doc page /focus
- Split JavaScript => Do this after asset reconciliation
- input.setCustomValidity() should block submission
- Replace symlinks in dist/ with copied files
  - Not trivial, see hk/copy-dist-artifacts
- Maybe a way to mark elements as keep before rendering?
  - This would also be up:fragment:loaded
  - Maybe a render option { useKeep: [....] }
    - With additional fragments to keep
    - Could be set both in initial render and in up:fragment:loaded
    - Would not require an [up-keep] on the other side
- Consider keepSelectors and noKeepSelectors
  - Now that [up-keep] no longer has a value?
    - Note that we do have [up-keep] hardcoded in multiple places
- Should we offer a way to manually add etags?
  - E.g. up.on('up:request:loaded', ({ response }) => { response.etag ||= sha1(response.body) })
  - Should this even be a default?
  => People can already set response.headers['ETag'], although that's not documented
- Matching in the region of ancestors should use subtree, not just descendants of the ancestor
  - In a form we should always be able to say form:has(:origin) .descendant, even if the origin turns out to be the form itself
    - But :has() does not work like that?
- [up-on-accepted-reload], { onAcceptedReload }
- [up-on-accepted-validate], { onAcceptedValidate } (aber hier brauche ich immer einen param)
- Consider removing all tagnames from public selectors
  - We already had to remove tag names of multiple-use attrs like [up-watch]
  - current
    - a[up-follow]
    - a[up-instant]
    - a[up-preload]
    - form[up-submit]
    - a[up-accept]
    - a[up-dismiss]
    - a[up-layer=new]
    - a[up-transition]    \__
    - form[up-transition] /
    - a[up-alias]
    - a.up-current
    - a[up-emit]
    - a[up-back]
  - deprecated
    - a[up-close]
    - a[up-drawer]
    - a[up-modal]
    - a[up-popup]
- Can layer objects nullify their { element, tether } once closed?
  => This may cause async JS to throw, if that JS wants to go through the layer shortly after closing
  => How about we do this 10 seconds after { onFinished } or so?
- It may be nice if guardEvents see the change's getPreflightProps().fragments
  - Either lazy with Object.definePropery() *or*, if we find that we're going to call getPreflightProps() anyway, just call it.
  - Can we memoize getPreflightProps() in FromURL ?
- up.validate() always submits to the first submit button
  => Is this a bug or a feature that we use the same behavior as submit would?
- When a layer reaches a close condition, should we still render [up-hungry][up-if-layer=any] elements in another layer?
  - Would be practical for flashes
  - Would be weird for API. Render events flying afterwards, we cannot communicate a RenderResult.
- When opening a new layer, we should update [up-hungry][up-if-layer=any] elements
  => This is not trivial as OpenLayer cannot work with multiple steps yet
  => Maybe we can do a "secret UpdateLayer pass" once the layer has opened.
    - We would need to merge RenderResult objects
- Custom attributes are not copied into [up-expand]
  - up.link.config.expandableAttributes ?
    - can also include regexp
      - maybe reuse matcher logic for badTargetClasses
    - what if you need to copy an individual class, not an attribute?
      - up:link:expanded event for custom handling?
 - up.link.config.expandableAttribute (attribute, element)
    - The element will always be a link
- Support custom expire times via Cache-Control: stale-while-revalidate
  - Print a warning when we're discarding any other Cache-Control header format
- Test and document [up-href]
- Polling should automatically restart when the tab is re-focused
- A validating fragment should get a .up-validating class
  - Group under up.feedback
  - Maybe we can have a private API
    - { feedbackLoadingClass: 'up-validating', feedbackActiveClass: false } ?
    - { feedback: { loading: 'up-validating', active: false } } ?
    - With history we have separate options { title, location }
- Consider renaming up.fragment.config.navigateOptions to just up.fragment.config.navigate
  - There is precedence in up.layer.config.overlay
- Firefox: Focus loss after disable is not always detectable synchronously
  - Force Repaint does not help
  - setTimeout() helps (wait for next render?)
  - We fixed this in up.form.disableWhile(), but it may still be broken for up.fragment.config.autoFocus options suffixed with "-if-lost"
- Consider whether validation requests should be background requests by default
  - We should also have [up-watch-background]
- Why does [up-validate=form] target the first form for some users?
  - https://github.com/unpoly/unpoly/discussions/474


Documentation
=============

- Should we offer a { placement: 'merge' } to merge children?
  - By selector?
  - How would that relate to recursive merging?
  - We don't need it for flashes, [up-on-keep] suffices here
    - [up-on-keep] can already support merging, morphing, etc.
      - However it's impractical to use it for regular fragment updates, you'd want .target:merge or something.

- Example for a custom overlay
  - Example in up:link:follow
  - https://github.com/unpoly/unpoly/discussions/473#discussioncomment-5665806

- Activating custom JavaScript
  - (Currently sitting in up.compiler())

- Rendering from strings
  - { document }
  - { fragment }
  - { content }

- Preloading
  - (Currently sitting in a[up-preload])
  - On Hover
  - Programmatically
    - Example: Preload next/prev

- Polling
  - (Currently sitting in [up-poll])

- Options and defaults
  - Everything is opt-in
  - Navigation enables a set of new defaults
    - Navigation options are the only opinioniated defaults that are opt-out
  - config options
  - auto-options
  - Most options in up.render() can also be set via an attribute
    - JavaScript options override HTML attributes
    - If we cannot serialize an option into an attribute, you can use `event.renderOptions` in up:link:follow, up:form:sumit
  - Explain that you can often override renderOptions in event handlers
    - This is already explained in /render-hooks

- Background requests
  - Background requests deprioritized over foreground requests.
  - Background requests also won't emit up:network:late events and won't trigger the progress bar.
  - Background requests are promoted to the foreground if they are a cache hit for a new, non-background request


Icebox / Tar pits
=================

- Allow late registrations of compilers and macros without priority
  => OK for compilers, but what about macros? They have an intrinsic priority (before all compilers)
- Consider whether up.validate() promises should wait until no more solutions are pending
  => We would need to merge RenderResult#target in some meaningful way
- Rename "finished" to "concluded"
- Should up:click set up.layer.current ?
  - It would be more convenient, but it's only relevant for popups or backdrop-less modals. This is rare.
- New onTransitioned() callback to take some load off from onFinished()
- Move scroll positions into state
  - This gets rid of the other up.Cache usage
  - This may mean we need to lose up.viewport.restoreScroll() and { scroll: 'restore' } and { saveScroll: true }
    - Losing { scroll: 'restore' } is super sad :(
  => Maybe revisit when the Navigation API is supported
- Improve polling in the background
  - It would be great to *not* have a timeout running while we're in the background or offline
  - It would be great to not wait up to 10 seconds when we're re-focused or regain connectivity
    - Are timeouts really paused or do they just not fire until re-focus?
    - Mobile Chrome seems to reload old tabs automatically, test this!
- Elemente mit [up-hungry][up-layer=any] müssten wir *eigentlich* auch austauschen, wenn wir einen neuen Layer öffnen
  - OpenLayer kann aber gar nicht mit mehreren Steps umgehen
- can badResponseTime be a function that takes a request?
  => Yes, but not trivially
- Consider using `Cache-Control: stale-while-revalidate=<seconds>` to indicate how long we can use a response
  - But it could be paired like this: Cache-Control: max-age=1, stale-while-revalidate=59
  - But then again we ignore Cache-Control for all other purposes
    - E.g. Cache-Control: no-store
    - E.g. Cache-Control: max-age
    - How would Cache-Control: no-cache (equivalent of max-age=0 must-revalidate) work in connection with up.fragment.config.autoRevalidate ?
  - Maybe do a bigger workover of Cache-Control?
- Do follow links with [target=_self]
- up:click forking could print a warning when an a[href] is instant, but not followable
- Is it weird that up.layer.affix appends to the first swappable element instead of the contentElement?
  - It's actually more like "appendableElement"
  - Maybe offer up.Layer#append
- Consider exposing up.layer.contentElement
- Do we want a shortcut macro for this:
      <input up-validate up-keep up-watch-event="input">
  - <input up-live-validate>
  - It's weird for users who don't target the input. They may expect to just override the event.
  - We would need to make keepable selectors configurable to include this one
- We could support this with more pseudo-class selectors like :form-group and :submit-button
  - :submit-button is hard to build origin-aware => It could just be a substitution with :is() / :-moz-any() & :-webkit-any()
  - :form-group is also supper hard to support in a selector like ".foo, :form-group, :bar" due to the way we hacked :has()
    - :has() is still behind a flag in Chrome and no Firefox support
- Introduce boundaries or "softly isolated zones"
  - The idea started with: Should fragment lookups with an { origin } within a form prefer to look within the form?
    - Also related to https://github.com/unpoly/unpoly/issues/197 , which would no longer work
      now that a form submission's orgin is the submit button instead of the form element
  - E.g. <div up-boundary>
    - Lookups within prefer to match within the boundary
    - It's a new fallback target
      - Also for errors
    - up.fragment.config.boundaryTargets = ['[up-boundary]', 'form', ':main']
    - Is this also controlled by { fallback }?
    - Maybe identification using [up-boundary=123]
      - But don't enforce this, it's not a great auto-target
    - Should this rather be [up-zone]?
      - If we ever make fully isolated containers we would call them frames
        - https://github.com/unpoly/unpoly/discussions/350
    - We could also offer :zone as a selector
    - Would we still offer { target: '.container .child' }?
      - Would we offer { target: ':zone .foo' }, since it's really the same as { target: '.foo' } ?
    - Is this a repetition of "fragment needs to know whether it is used as component or main target"?
      - We would need to fix infinite looping in expandTargets()
      - It would be nice to disable history in a zone
        - but then it's not usable as a main target
        - Disable history in a container?
          - It's weird to nest multiple containerish elements
        - => This is really already solved through { history: 'auto' }, which only updates history if updating :main
- Rendering: allow { scrollBehavior: 'smooth' } when we're not morphing
  - Could we even allow this *when* morphing?
- New up.render() options to clone from a template
  - { documentTemplate }, { fragmentTemplate }, { contentTemplate }
  - Separate doc page "Rendering from local content"
  - Fail when template cannot be found
  - But what if I really need to re-use an existing element that is then placed back into the body, like in WB?
- Consider implementing an abortable up:navigate event
  - This would be the first feature that goes beyond "navigation is just a preset"
  - People might expect history changes to trigger this
  - Maybe think about this more
- Replace up.hello() and up.script.clean() with MutationObserver()
- Do we want to serialize all actions in a form?
  - up-sequence="form"
  - This would need to abortable on submit => Would be handled by existing { solo: 'target' } IF there is a request
  - This would need to abortable on form destroy => Would be handled by existing { solo: 'target' } IF there is a request
  - => This would need to be handled by up.Queue, or else there would be nothing to abort
  - => It is not sufficient to have up.form.sequence(..., callback)
  - => We would need to do something like { sequence: ElementOfSubtree }
  - => Before picking a new request, make sure no existing request exists
  - What about our old idea: { order: 'abort target', order: 'abort form', order: 'after form', order: 'after selector' }
      => How to say "after ElementObject" ?
  - Who would fetch the element that is 'form' or 'selector'?
      => up.Change.UpdateLayer#getPreflightProps(), which already loads targetElements()
  - What would we do if both was given, e.g. { solo: true, sequence: 'form' }
    - Do we forbid the combination?
    - Do we first abort, then do a squence?
    - Do we first wait, then abort? => I think this, but this also means our { solo } handling is in the wrong place. It must move to the queue.
  - Does { sequence: 'form' } also queue local content, like { solo } ?
   - We could do something like up.LocalRequest, but then local updates would no longer be sync!
   - We could not support { sequence } for local updates => YES
  - What about cached content with { sequence }?
    - We could do queue.asapLocal() which immediately executes unless there is { sequence }
  - How does queue resolve a sequence?
    - Before dispatching a request with { sequence }
    - Check if we have *current* requests with { sequence }
    - If any of the other requests's sequence contains our *or* if any other sequence is contained by ours, don't dispatch
- Guard Events for Rendering could have a Promise for "done"
  - Is this better than setting event.renderOptions.onFinished()?
    - Yes, because onFinished may not fire for fatals or prevented up:fragment:loaded
  - How would this work together with future up.RenderRequest?
  - How would this work together with "local changes are sync"?
- Consolidate [up-validate], [up-switch] and [up-watch] into a tree of form dependencies
  - This way we can selectively disable parts of the form
- Functionality that checks for isDetached() should probably also check for .up-destroying
- Improve `{ focus: 'keep' }` so it focuses the former form group if we lose focus
  - This may be easier said than done
    - we would need to remember the original form group before the swap in the FocusCapsule
    - re-discover the form group in the new HTML
    - check that the form group is a closer match than target-if-lost
    - come up for a better name for the option (target-if-lost)
- New event up:request:solo ?
- Consider delaying appending of new layer elements until we have constructed all children https://github.com/unpoly/unpoly/discussions/314
- Publish { onQueued }
  - We're currently only using onQueued to get the request of a rander job, so we can abort it
    - This may be removed!
  - More canonic would be if RenderJob had an abort() method
- Wir aborten bereits laufende [up-validate] wenn das Formular submitted, wird, aber laufende Watcher-Delays warten können danach noch Dinge tun
  - Wie wäre "submit stoppt das delay"?
  Evtl. Warnung ausbauen: "Will not watch fields without [name]"
- [up-emit] auf Buttons erlauben
- Beim Schließen des Modals prevented jemand up:layer:dismiss, und dann steht "Abort Error: Close event was prevented" in der Konsole.
  - Wollen wir das schlucken?
  - Zumindest bei ui-elementen wie [up-dismiss] ?
- DestructorPass sammelt zwar Exceptions, aber wirft am Ende. Wer fängt das dann? Der Wunsch wäre, dass das drumrumliegende up.destroy() noch zu Ende läuft, dann aber up.CompilerError wirft.
- ConstructorPass sammelt zwar Exceptions, aber wirft am Ende. Wer fängt das dann? Der Wunsch wäre, dass das drumrumliegende up.render() oder up.hello() noch zu Ende läuft, dann aber mit up.CompilerError rejected.
- Update "Long Story" Slides with new API
- Doc page about "Fragments from local HTML"
  - link from [up-document], [up-fragment], [up-content], { document, fragment, content }.
- Warn when registering compiler in [up-] namespace
- Consider documenting errors
  - But what would be the @parent ?
  - up.CannotCompile
  - up.CannotMatch
  - up.Offline
  - up.AbortError
    - has { name: 'AbortError' }
- Should the old "clear" be "expire" or "evict"?
  => We really want to push our new defaults for both
  => I think it should be "expire". Most users set a lower expire time.
- remove up.util.flatMap() => No, we need it to flatMap array-like objects (e.g. arguments)
  - Do we want to move to saveState() / restoreState()?
    - I think we want to keep the [up-focus] and [up-scroll] options separate.
      - E.g. we want to focus the main element, but reset scroll.
      - This could also be fixed by revealSnap ?
    - These are eerily similar:
      - https://unpoly.com/scrolling
      - https://unpoly.com/focus
      - The -if-lost suffix can only pertain to focus
    - What would be the name for such an attribute?
      - [up-spotlight]
      - [up-viewport] (classes with [up-viewport]
      - [up-highlight]
      - [up-locus]
      - [up-present]
      - [up-light]
      - [up-shine]
      - [up-state]   (seltsam: up-state=".element")
      - [up-point]
      - [up-pinpoint]
      - [up-attention]
      - [up-focus] also scrolls?
      - [up-show]
      - [up-view]
    => I think power users want to control this separately
    => Also we need to call it at different times
    => Also the auto options work differently, e.g. if there is an [autoscroll] element in the new fragment
    => We might offer a shortcut like [up-view] and [up-save-view] as a shortcut to set both at once
- Replace up.rails by accepting methods / conform from data attributes in options parser
  => This wouldn't work in scenarios where both Rails UJS and Unpoly were active
- No longer send X-Up-Location with every response
  => No we should keep sending it, as this excludes redirect-forwarded up-params
- Consider reverting: up:request:late: Consider the time a promoted request was loading in the background
  => For this we would need to track when a request was promoted to the foreground
- Do we trigger onFinished when preloading?
  => No, users can use the promise or onLoaded()
- Reconsider how many lifecycle callbacks we want to parse from links
  - Benchmark up.link.followOptions()
    - console.time('parse'); for (var i = 0; i < 10000; i++) { up.link.followOptions(link) }; console.timeEnd('parse')
    - VM481:1 parse: 1091.6689453125 ms
    => It takes 0.1 ms
    => This is not a performance issue
- Find a scenario where it's better to read the etag from [etag] instead of response.etag
  - This should not matter for revalidation after a swap
  - When reloading an arbitrary fragment, an earlier response may not be available
- Test if browsers honor cache keys for XHR requests
  - Yes, it honors Cache-Control: max-age=...
  - We can override it for fetch()
- With our long cache eviction background tabs could hog a lot of memory
  => No, since we also limit the number of cache entries to the cache never exceeds some MB
- up.link.cacheState(link) to people can build .offline classes themselves
  => Users can already use up.cache.get
  => We could clean up Request#state and publish this for more goodness
  - Return any known up.Respone for the given link
    - It already has useful properties { evictAge }, { expireAge }?
    - We may eventually offer up.Response#revalidate()
  - Return null while the request is in flight
    => Or do we want an up.CacheState that also returns here?
  - It will be hard to do implement this without actually calling up.link.follow() and up.render(), since e.g. the target choice is hard and part of the cache key
    - Make this an early return in up.Change.FromURL, like with options.preload
- Test if we can preserve element.upPolling (or at least the { forceState }) through up.reload({ data })
  => It's hard since there's the case that the server no longer responds with [up-poll], but we want to keep polling when forceStarted
- Should we delete options.data if a fallback is loaded?
  - This would also be true for other options, like a selector in focus/scroll
  - We would sometimes need to guess, e.g. { focus: ':main' } may also be a good default for a fallback
  => I think this is not a real case since we're not going to use { data } together with { fallback }
- Consider moving [up-etag], [up-time] and related functions to up.protocol
  => While there's some merit to that, we would need to rename up.fragment.etag() to up.protocol.fragmentETag() etc.
  => What's with up.fragment.source()?
- [up-hungry][up-placement]
  => This only makes sence when we have some form of deduping
- What do we log when no replacement target is available? => up.render() throws an error
- Reconsider why we don't call onRendered() for empty updates
  - Pro: Call it for every pass
    - We do still update scroll and focus, even for empty responses
      - Maybe even location?
    - We will update more in the future with head merging
    - There is no other hook for "after $things changed"
      - But nothing changed after an empty render? At least not in the DOM.
    - We can simplify docs for onRender and up-on-render
  - Con: Keep it like now
    - If we render without fallback we still need to check if up.RenderResult#fragment is defined
    - It will be super rare that the first render pass does not return anything.
      - However, we would *always* get empty onRendered calls for revalidation
  => Keep it, but improve docs
- Maybe ship empty unpoly.es5.js und unpoly.es5.min.js that just say to use ES6
  => Not worth the hassle
- Does it make sense to have onKeep in render options?
  => I find it confusing to have both { onKeep } and { useKeep } in the same functions
  => Maybe we can find a way to merge it with { useKeep }
- Consider up.layer.config.isolateLabels
  => Wait until there is a use case
  => Users can always just intercept `click` themselves
- Consider [aria-haspopup]
  => This would require a compiler automatically setting this for [up-layer=new] links (and all shortcuts)
  => Don't do this for performance. Delegate to apps.
- Should config.autoCache allow to cache error responses? => That's a hard question to answer
- Get rid of response.request (which contains references to a layer), response.xhr ?
  => This is hard because up.cache.expire() expires requests, not responses. Hence up.Response delegates { expired } to its response.
- Can we allow await in Callbacks?
  - new AsyncFunction()
  - This would not work trivially with the postprocessing in up.NonceableCallback
    - At least we would need to detect use of `await`
- Returning 'auto' for default would be a good way to override configs with exceptions
  - request.url.endsWith('/edit') ? false : 'auto'
  - Since this already the 'autoMeans' config we cannot return 'auto' here
- Replace u.flatten() with Array#flat()
  => We keep array-related utils because ours work with non-Array lists and iterators
- Replace u.flatMap() with Array#flatMap()
  => We keep array-related utils because ours work with non-Array lists and iterators
- Now that compilers have the meta arg it should be possible to give up.RenderResult new { request, response } props
  => Wait until we have a use case for this
- Test synthetic ETag
  - I think up.Response#headers is not documented
  => There's a simpler way to this documented in https://v3.unpoly.com/up:fragment:loaded#example-discarding-a-revalidation-response
- When rendering uses <up-wrapper> elements, do we set [up-time] and [up-etag] on these wrappers?
  - We cannot really do better here, since up-wrapper children cannot be reloaded
- Move `isRenderableLayer()` logic to LayerLookup
  - Consider having LayerLookup#getAll() throw if results are blank
  => We have a lot of code / specs that work with closed layers and sometimes normalize the layer through up.layer.get(), and now get undefined.
     All of this code would fail with this change.
- Terminology for notifications
  - [up-alerts] (Bootstrap, Tailwind UI, MUI, negative connotation in Material etc.)
  - [up-flashes] (Rails, Primer [although they also use Toast])
  - [up-notifications] (Bulma, good for both success and failure, long word, may also mean passive notifications)
  - [up-messages] (too generic)
  - [up-snackbars] (Material, uncommon)
  - [up-toasts] (uncommon, also often a "boxy" kind of notification)
  - [up-callouts] (Foundation, uncommon)
- Do we want to support changing the renderOptions in up:fragment:hungry? In that case { renderOptions } need to be { nonHungryRenderOptions } or something
  => We can rename the options then. If we do support this we would no longer have { renderOptions }, we would use { targetRenderOptions, hungryRenderOptions }
- Instead of ResponseDoc#commitSteps(), could up.radio.hungrySteps() just re-compress steps with uniqBy(steps, 'selector')?
  => There is an edge case where different selectors could match the same elemnent
- Introduce up:history:restore event
  - With { renderOptions }
  - Note that we used to emit this event in the past, so fix unpoly-migrate
  - Preventable for custom history handling
  => We already have this in up:location:restore
- Reduce render callbacks
  - Motivation
    - It is too hard to do something when the render job settles. There is onRendered, onFinished, onError etc. However there are always cases that are covered by no callback.
    - We need to maintain and communicate two styles of callbacks
  - Expose { renderJob } on guard events
  - New event up:fragment:render / [up-on-render] that gets { renderOptions, renderJob }
    - There is no need to offer { onRender }, as the return value of up.render() is already a RenderJob
  - Deprecate callbacks
    - { onFinished }
    - { onRendered }
  - Update docs
    - https://unpoly.com/render-hooks#preventing-a-render-pass
    - https://unpoly.com/render-hooks#changing-options-before-rendering
  => No, don't do that.
    - While we can return a RenderJob, it's inner promise is not yet set to a RenderResult
    - We also have { onFinished } in other places, e.g. in dismiss() and accept(). There is no CloseJob or similiar there.
- Do we need a way to show a spinner for a while?
  - Maybe, but you can already do in CSS: .container:has(.up-active) .spinner { ... }
- Do we really need to abort requests in LinkPreloader?
  => Yes, aborted requests can be seen in the network tab. We also found a shorter way to write it. 

- Allow the [wants-menu] compiler to be implemented without JS
  - We would need something like [up-defer][up-unless-already-there] 
    - Or even a follow option?
    - Should we just check, or "only revalidate"?
    - Or should it be the default behavior?
    => This is really hard to do right
- Do we dare to restore history using up.render() instead of up.navigate()?
  => We override as much as want to re-use
- Do we want [up-skeleton=<string>] ?
  - We would need to decide what to do within overlays
    - Show skeleton within overlay?
    - Animation
    - Different skeleton
  => Later
- Consider renaming [up-feedback] to something else
  - Maybe always have classes?
    - This would mirror the exception of background, which is also false by default
  => Delay this until we need the setting for something else
- Consider offering a { renderLayer } prop to guard events
  - This would make it easier to e.g. auto-add a preview for new layers in up:link:follow
  - Do we need to give a better name to { layer } so people understand it is not the origin layer?
    - There is { targetLayer }, but that could be confused with event.target (which happens to be the origin layer!)
    - { changeLayer }
    - { updateLayer }
    - { targetedLayer }
    - { planLayer }
    - { insertLayer }
    - { renderLayer }
  - Update docs for guardEvents
    - up:link:follow    [yes]
    - up:form:submit    [yes]
    - up:link:preload   [yes]
    - up:deferred:load  [makes no sense]
    - up:form:validate  [makes no sense]
    - up:fragment:poll  [makes no sense]
  => No, this becomes very complicated very fast
- Consider whether { skeleton } is flexible enough for public API
  - We cannot show skeletons with multiple fragments
    => But we're fine with that limitation for [up-transition]
  - We cannot control the parent
    => But we're fine with that limitation for [up-transition]
  - [NO!] Maybe we could offer a callback form, e.g. <a href="/foo" up-preview="preview.showSkeleton('#foo', '#skeleton')">
    - We already support this for JS functions
    - To also support named previews we would need to look for "preview." in the string
    => All Preview methods would need to support selectors, which means we would need to think about layers
- Deprecate flatten() in favor of Array#flat(), flatMap() in favor of Array#flatMap()
  - No, we use this a lot of non-array values, in particular NodeLists

- Do we want to keep { feedback }, [up-feedback] as an option, or applied it always?
  - It's already a navigation default
  - For programmatic up.render() calls, it's already activated by passing { origin }
  => We can decide this later
- Do we want Preview#affix?
  - Check if it helps with the demo
  => Not for now. I'm myself moving away from affix.
- Previews should know about placement?
  - Unsure. Later.
- Rename "preview" to "teaser"?
  - => It's too confusing with { placeholder }
- Test that re-attaching an helloing an destroyed element can be compiled again
  - This is not trivial because destructors would need to clean up Element#upAppliedCompilers
- Move config.progressBar to up.status ?
  - Is it weird that up:network:late and up:network:recover events are still in the up.network package?
  - We also keep [up-disable] under up.form
  => Keep it there
- What if there is user input in the reverted preview?
  - E.g. we want to show the "Edit task" input instantly, but keep input as we revert with the real thing
    => Preview fns could manipulate event.renderOptions
    => If we already have a template, we could just render a template with [up-fragment] and not make a request
- Think whether we want to support JSON => HTML preprocessing
  - This would something like <a href="/foo.json" up-convert-with-template="#foo">
  - This would need to disable history
- Always run callbacks with `<script>` so users can use `strict-dynamic`
  => This would work, but we would violate the spirit of a CSP, even with strict-dynamic
  => This would execute an attacker's [up-on] callback
  => Don't do this!

- Also have a up:field:switch (up:form:switch?) event
  - preventable? maybe
  - example: [readonly]
  - Expose the dependent container as { target }
  - expose { values } array, or just { field }
  => No, we need to selector to detect new fields

