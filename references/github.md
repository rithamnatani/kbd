Two working concepts to avoid unintended home row mod (HMR) activation are currently (2025-06) around.

fast typing layer: disable home row mods, while typing fast, by automatic switching to a layer without home row mods. (historical thread #502 )

key-timing: After 3 key presses within eg. 140ms disable home row mods. ( from @argenkiwi)

here are some implementations (from the past?!)

    key-timing: Skip home row mods while typing #1455 enforces use of tap-hold-release

    advanced-sample: https://github.com/jtroo/kanata/blob/main/cfg_samples/home-row-mod-advanced.kbd uses tap-hold-release-keys

    one-shot typing layer: Home row mods and automatic fast typing layer #1425 uses tap-hold

I DISSMISSED my own one-shot typing layer, because one-shot handling ran into issues as my typing speed increased.

Therefore I started my "key-timing" vs "typing-layer" investigation.

In theory

    key-timing: needs two random key presses before HRM deactivation
    typing-layer: needs one letter key tap before HRM deactivation

In practice chording/roling the first letters will delay the activation of both methods.

I still DISSMISSED key-timing - because

    it just produced more errors for me
    non of the (optional) features from below are easy to achiev
    most importantly, I found no way to fix nasty bigramms/roles

I tried chords like this to fix them, but it didn't work out

(deftemplate orderedchord (first second)
  (switch
    ((input-history real $first  2)) (multi $first $second) break
    ((input-history real $second 2)) (macro $second $first) break
    () XX break
  )
)
(defchordsv2
  (f d) (t! orderedchord f d) 50 all-released  (numbers) ;; prefer bigramm over homerowmods
)  

Whereas typing-layers offer tap-hold-release-keys as a fix for nasty bigramms.

But I DISSMISSED the advanced-sample implementation, too - because it blocked too much (on the same hand) and too little (on the other hand)

So I made a new "typing-layer" approach close to advanced-sample but with following improvments/options

    ditch the whole right hand / left hand idea to stop overuse of tap-hold-release-keys
    individual timings for each of your fingers - consider finger speed and mod relevance - that way ALT and META missactivations are gone (start with higher timeout values as a beginner)
    exit "typing layer" when hitting space, because upper case is more likely after space. This allows experimenting with longer on-idle timeouts
    use tap-hold-release-keys only to fix really problematic bigrams, but prefer plain tap-hold whenever possible.
    (optional) temporary key deactivation of caps, numbers, ... or any other key you don't want to hit by accident
    (optional) re-add tap-hold or double-tap to the typing layer if they are executed that fast
    (optional) re-add home row shift (with higher timeouts) for CamelCase (tap-hold  0 310 f lsft) to the typing layer
    (optional) use (unshift) in typing-layer for 'shifted' roles into 'proper case' roles

These are all my bigramm fixes I started with: only fd and df are same hand fixes, while dh, dk and d; are opposite hand fixes

f (t! homerowmodfiltered 0 160 f lsft (d))
d (t! homerowmodfiltered 0 250 d lctl (f h k ;))

revisiting them after two months, and I deleted them all because I learned the timings

Here is the final implementation
Since I had an fast typist i.e. @rainmeower as tester see this config should be 'speedproof'

(defcfg
process-unmapped-keys yes 
)
;; timeings should work fine for 40 to 80 words per minute; if you are slower or new to home row mods increasing them might help in getting started   

(defvirtualkeys
  to-base (layer-switch base)
)

(defvar
  tot 220  ;; tot=time out tap
)

(deftemplate homerowmod (timeouttap timeouthold keytap keyhold)
	(tap-hold $timeouttap $timeouthold 
		(multi $keytap  @.tp) 
		$keyhold
	)
)

;; homerowmodfiltered is a way to force problematic bigrams to become letters not mods
(deftemplate homerowmodfiltered (timeouttap timeouthold keytap keyhold typinglist)
  (tap-hold-release-keys $timeouttap $timeouthold
    (multi $keytap  @.tp)
    $keyhold
    $typinglist ;; Activates $keytap early if a key within $typinglist is pressed before hold activates.
  )
)

(defalias
  ;; call @.tp whenever you want to enter fast typing-layer
  .tp (switch
        ;;(lsft rsft) XX break ;; optional skip typing-layer activation for upper case keys might solves some FJ problems
        ()  (multi
              (layer-switch typing)
              (on-idle 55 tap-vkey to-base) ;; timeouts from 30 to 100 are wide spread
            ) break
      )
  .spc-typing   (multi (layer-switch base) spc) ;; expilcitly leave typing-layer when hitting `space` - this allows experimenting with higher idle timeouts
)

(defsrc )

(deflayermap (base)
 ;; define home row mods (they act as typing-layer triggers, too )
 f (t! homerowmod $tot 160 f lsft)
 j (t! homerowmod $tot 160 j rsft)
 d (t! homerowmod $tot 300 d lctl)
 k (t! homerowmod $tot 300 k rctl)
 s (t! homerowmodfiltered $tot 400 s lalt (a)) ;; example fix of the 'as' role 
 l (t! homerowmod $tot 400 l ralt)
 a (t! homerowmod $tot 450 a lmet)
 ; (t! homerowmod $tot 450 ; rmet)
 ;; define each letter as typing-layer trigger
 q (multi q @.tp) w (multi w @.tp) e (multi e @.tp) r (multi r @.tp) t (multi t @.tp) y (multi y @.tp) u (multi u @.tp) i (multi i @.tp) o (multi o @.tp) p (multi p @.tp) g (multi g @.tp) h (multi h @.tp) z (multi z @.tp) x (multi x @.tp) c (multi c @.tp) v (multi v @.tp) b (multi b @.tp) n (multi n @.tp) m (multi m @.tp) 
)

(deflayermap (typing) 
 a a b b c c d d e e f f g g h h i i j j k k l l m m n n o o p p q q r r s s t t u u v v w w x x y y z z 
 caps XX ;; key deactivations of caps,... are optional
)

'd be happy to hear from your adaptations.

In this version hold-for-duration is used instead of on-idle. While being simpler tap-hold-release-keys/homerowmodfiltered now has issues and only homerowmod(wich does note use tap-hold-release-keys) should be used.

Since it does not rely on the idle state, features like caps-words now work.

(defcfg
process-unmapped-keys yes 
)
;; timeings should work fine for 40 to 80 words per minute; if you are slower or new to home row mods increasing them might help in getting started   

(defvirtualkeys
    fast    (layer-while-held typing)
)

(defvar
  tot 220  ;; tot=time out tap
)

(deftemplate homerowmod (timeouttap timeouthold keytap keyhold)
	(tap-hold $timeouttap $timeouthold 
		(multi $keytap  @.tp) 
		$keyhold
	)
)

;; homerowmodfiltered is a way to force problematic bigrams to become letters not mods
(deftemplate homerowmodfiltered (timeouttap timeouthold keytap keyhold typinglist)
  (tap-hold-release-keys $timeouttap $timeouthold
    (multi $keytap  @.tp)
    $keyhold
    $typinglist ;; Activates $keytap early if a key within $typinglist is pressed before hold activates.
  )
)

(defalias
  ;; call @.tp whenever you want to enter fast typing-layer
  .tp (hold-for-duration 55 fast)  ;; timeouts from 30 to 100 are wide spread
  .spc-typing   (multi (on-press release-vkey fast) spc) ;; expilcitly leave typing-layer when hitting `space` - this allows experimenting with higher idle timeouts
)

(defsrc )

(deflayermap (base)
 ;; define home row mods (they act as typing-layer triggers, too )
 f (t! homerowmod $tot 160 f lsft)
 j (t! homerowmod $tot 160 j rsft)
 d (t! homerowmod $tot 300 d lctl)
 k (t! homerowmod $tot 300 k rctl)
 s (t! homerowmodfiltered $tot 400 s lalt (a)) ;; example fix of the 'as' role 
 l (t! homerowmod $tot 400 l ralt)
 a (t! homerowmod $tot 450 a lmet)
 ; (t! homerowmod $tot 450 ; rmet)
 ;; define each letter as typing-layer trigger
 q (multi q @.tp) w (multi w @.tp) e (multi e @.tp) r (multi r @.tp) t (multi t @.tp) y (multi y @.tp) u (multi u @.tp) i (multi i @.tp) o (multi o @.tp) p (multi p @.tp) g (multi g @.tp) h (multi h @.tp) z (multi z @.tp) x (multi x @.tp) c (multi c @.tp) v (multi v @.tp) b (multi b @.tp) n (multi n @.tp) m (multi m @.tp) 
)

(deflayermap (typing) 
 a a b b c c d d e e f f g g h h i i j j k k l l m m n n o o p p q q r r s s t t u u v v w w x x y y z z 
 caps XX ;; key deactivations of caps,... are optional
)

24 replies
@gerhard-h
gerhard-h
on Sep 9, 2025
Author

Have you tried if ctl + shift + s if it works, when pressed in that order?
I made a very similar solution #1647 (reply in thread) in a discussion with @nyxmeowmeow
I remember I liked its home row mod performance (no difference compared to "typing layer" for me) during testing.
But it spams nop0 into key-history, which annoyed me more than having to handle an extra layer
@musjj
musjj
on Sep 9, 2025

Yep, Ctrl + Shift + S works just fine for me.

    But it spams nop0 into key-history

Yeah this is definitely a huge drawback for key-history users. The problem is that for some reason you can't just do:

(defvirtualkeys typing XX)

If this works, key-history will no longer be polluted. But I guess it's a limitation of Kanata's current implementation.
@musjj
musjj
on Sep 9, 2025

I figured out a really goofy workaround for key-history users: Simulator link.

The gist is that the state of a virtual key can only be tracked if it contains a "real" action. So to make it work we smuggle a virtual key action (that is actually no-op), inside of the virtual key that we want to track:

(defvirtualkeys
  bar XX
  foo (on-press toggle-virtualkey bar))

Now, we can properly track the state of foo through (input virtual foo). But this time it's truly side-effect-free (no more key-history pollution)!

The fact that this workaround works means that the mechanism already exists underneath Kanata, it just needs to be better exposed.
@gerhard-h
gerhard-h
on Sep 12, 2025
Author

When testing this virtual-XX concept, I realized that short hold durations like 55ms here worked much better in avoiding unintended SHIFT activation than longer ones (which seems counterintuitive).
Short hold durations also mean, no extra release-handling of spc or layer keys is needed.
I also use tap-hold over tap-hold-release to avoid unintended Non-Shift-Mod activations.

I get this warning (with my production config I get it 6 times) but I believe it's safe to ignore.
[WARN] XX within defvirtualkeys is likely incorrect. You should use nop0-nop9 instead.
@jtroo is this virtual-XX concept worth to be mentioned in the docs as a way to safe state kanata without affecting the keyboard state?

camelCase works out of the box, also chords, zippychords, keyhistory are not affected. Looks like a really good improvement.

EDIT PRE1.10 with [v1.10.0-prerelease-1] the code got simpler as you don't need the virtual toggle key anymore (and it stopt working )

(defcfg
process-unmapped-keys yes 
)

(defvirtualkeys
  fasttypingstate XX
  ;; PRE1.10 fasttypingtoggle (on-press toggle-virtualkey fasttypingstate)
)

(defvar
  tot 220  ;; time out tap
)

(deftemplate type (action)
  (multi
    (fork
       (hold-for-duration 55 fasttypingstate)
       ;; PRE1.10 (hold-for-duration 55 fasttypingtoggle) ;; short timeouts seem to work best 
      XX
      (lctl rctl lalt ralt lmet rmet)
    )
    $action
  )
)

(deftemplate homerowmod (timeouttap timeouthold keytap keyhold)
  (switch
    ((input virtual fasttypingstate)) (t! type $keytap) break
    () (tap-hold $timeouttap $timeouthold
         (t! type $keytap)
         $keyhold
       ) break
  )
)
(defsrc )

(deflayermap (base)
 ;; define home row mods
 f (t! homerowmod $tot 160 f lsft)
 j (t! homerowmod $tot 160 j rsft)
d (t! homerowmod $tot 300 d lctl)
 k (t! homerowmod $tot 300 k rctl)
 s (t! homerowmod $tot 400 s lalt)
 l (t! homerowmod $tot 400 l ralt)
 a (t! homerowmod $tot 450 a lmet)
 ; (t! homerowmod $tot 450 ; rmet)
 ;; define each letter
q (t! type q) w (t! type w) e (t! type e) r (t! type r) t (t! type t)
y (t! type y) u (t! type u) i (t! type i) o (t! type o) p (t! type p)
 ;; a (t! type a) s (t! type s) d (t! type d) f (t! type f) 
g (t! type g h (t! type h)
;;  j (t! type j) k (t! typek) l (t! type l)
 z (t! type z) x (t! type x) c (t! type c) v (t! type v) b (t! type b) 
 n (t! type n) m (t! type m) 
)
