 A try on urob's timeless home row mods for kanata

urob has cracked the code for good and usable home row mods, essentially circumventing many of the pittfals Precondition has wrote about in his amazing piece on home row mods.

urob's magic uses a few ingredients that ZMK makes possible to tweak on hold-tap functionality and have inspired many to port it to other firmwares (recently, Pascal Getreuer has ported it to QMK and also did a great job compiling the "equivalences" of these features between ZMK, QMK and even Kanata here.).

I use these settings on my ZMK boards, and was eager to use them while typing on my MacBook as well. And now, I believe I've managed to replicate in Kanata all the functionality that makes urob's timeless home row mods so amazing.

In his config urob uses a bunch of macros in his config, so I've translated it into vanilla ZMK:

hrml: hrml { // urob's home row mods left
	
	// this is usual
	compatible = "zmk,behavior-hold-tap";
	label = "HRML";
	bindings = <&kp>, <&kp>;
	#binding-cells = <2>;
	quick-tap-ms = <175>;
	tapping-term-ms = <280>;
	
	// here's where the magic begins
	flavor = "balanced";
	hold-trigger-key-positions = <RIGHT_SIDE_KEYS THUMB_KEYS>;
	hold-trigger-on-release;
	require-prior-idle-ms = <150>;
};

urob's does a great job of explaining the why of each of these parameters, so I'll spare you of the details here.

I've used the above config snippet to isolate the four items we need to fully replicate on [[Kanata]] to achieve similar functionality:

    flavor: "balanced"

    hold-trigger-key-positions

    hold-trigger-on-release

    require-prior-idle-ms

Recently, Kanata merged the tap-hold-release-tap-keys-release variant (docs) of its tap-hold action. This, essentially replicates items 1 to 3 of the list if we configure it this way:

(tap-hold-release-tap-keys-release          ;; = zmk's "balanced" flavor
	$tap-repress-timeout                    ;; = zmk's quick-tap-ms
	$hold-timeout                           ;; = zmk's tapping-term-ms
	$tap-action                             ;; = tap binding
	$hold-action                            ;; = hold binding
	$tap-trigger-keys-on-press              ;; ≅ hold-trigger-key-positions (but without the hold-trigger-on-release)
	$tap-trigger-keys-on-press-then-release ;; = hold-trigger-key-positions with hold-trigger-on-release
)

So the only piece missing was the zmk's require-prior-idle.

That needed some creativity to get it working, but I've managed with some tweak to gerhard-h's solution on Kanata's Issue #1425.

That works by using a virtualkey, an alias, and a special layer that replicates the base layer, without without the mods, just typing key. Essentially, it uses an on-idle command with the virtualkey to move to a no-mods typing layer and switches back to the default layer with tap-holds after a the require-prior-idle timeout elapses.

So here's a simplified version of my config that uses this logic:

(defvar
  ;; variables for timings in ms using ZMK-like names
  tapping-term 300                       ;; A higher number allows for one handed modded keys reducing the chance of false-positives when typing fast
  quick-tap-ms 200                       ;; same as ZMK ($tap-repress-timeout in Kanata docs)
  require-prior-idle 250                 ;; same as ZMK (not used in vanilla kanata, but used as a workaround in this config)  
  
  ;; keys for tap-hold-release-tap-keys-release templates
  left-side-keys   (q w e r t z x c v b) ;; these keys will always make the precedent same hand tap-hold-release-tap-keys-release output tap.
  home-row-left-keys  (a s d f g)      ;; home row mods + z for fn  - these will output a tap if they're released after the first key
)

(deftemplate lhrm (tap-key hold-key) ;; these are home row mods for the left hand
	(tap-hold-release-tap-keys-release
		$quick-tap-ms
		$tapping-term 
		(multi $tap-key @tap) 
		$hold-key 
		$left-side-keys
		$home-row-left-keys)
)
;; must replicate for the right side, shortened here for brevity

(defalias
	
    tap (multi 
        (layer-switch nomods)                             
        (on-idle $require-prior-idle tap-virtualkey to-base)
    )

    a (t! lhrm   a lctrl  )
    s (t! lhrm   s lalt   )
    d (t! lhrm   d lsft   )
    f (t! lhrm   f lmet   )
      
(defvirtualkeys
	to-base (layer-switch def)
)

;; source
(defsrc
  q w e r t     y u i o p
  a s d f g     h j k l ;
  z x c v b     n m , . /
       lmet spc rmet 
)

;; keymap
(deflayer def
  q  w  e  r  t       y  u  i  o  p
 @a @s @d @f  g       h  j  k  l  ' 
  z  x  c  v  b       n  m  ,  .  ;
			    _ _ _
)

(deflayer nomods
  q  w  e  r  t       y  u  i  o  p
  a  s @d  f  g       h  j  k  l  '  ;; shift keys are still here to except it from `require-prior-idle` timmings
  z  x  c  v  b       n  m  ,  .  ;
			    _ _ _
)

Here's the full config if you're curious.

Many thanks to the many people that messed with the code on the web and specially on kanata's GitHub. This is possible thanks to them!


