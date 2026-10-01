# AdMob integration — placements, policy and wiring

Companion to `mobile-app.html`. That file carries the **placement and policy layer**:
every ad slot is tagged `data-ad-placement`, and the `ADS` functions in its script
decide when a request is allowed. This file is the part you take to the native app.

## Read this first

AdMob is a **native SDK**. It ships for Android and iOS, not for a browser page, so
the `ADS` block in `mobile-app.html` does not load ads — it models the placement
rules and the entitlement gating, and renders the slots so you can see where
everything lands. The real loaders are the Kotlin snippets below.

If you ever want ads in a desktop or mobile *web* build, that is **AdSense**, not
AdMob. Do not point an AdMob publisher ID at a web page.

## 1. Ad units

Swap every test ID for your own before you publish. Google suspends accounts that
serve or click live ads during development.

An **app ID** uses `~` and goes in the manifest / `Info.plist`.
An **ad unit ID** uses `/` and goes in the ad request.

### Test units — use these while building

| Format | Android | iOS |
| --- | --- | --- |
| App open | `ca-app-pub-3940256099942544/9257395921` | `ca-app-pub-3940256099942544/5575463023` |
| Anchored adaptive banner | `ca-app-pub-3940256099942544/9214589741` | `ca-app-pub-3940256099942544/2435281174` |
| Fixed size banner | `ca-app-pub-3940256099942544/6300978111` | `ca-app-pub-3940256099942544/2934735716` |
| Interstitial | `ca-app-pub-3940256099942544/1033173712` | `ca-app-pub-3940256099942544/4411468910` |
| Rewarded | `ca-app-pub-3940256099942544/5224354917` | `ca-app-pub-3940256099942544/1712485313` |
| Rewarded interstitial | `ca-app-pub-3940256099942544/5354046379` | `ca-app-pub-3940256099942544/6978759866` |
| Native | `ca-app-pub-3940256099942544/2247696110` | `ca-app-pub-3940256099942544/3986624511` |
| Native video | `ca-app-pub-3940256099942544/1044960115` | `ca-app-pub-3940256099942544/2521693316` |

Sample app IDs: Android `ca-app-pub-3940256099942544~3347511713`,
iOS `ca-app-pub-3940256099942544~1458002511`.

### Units this app uses

| Key | Format | Slot in `mobile-app.html` |
| --- | --- | --- |
| `home_banner` | Anchored adaptive banner | `#adHomeBanner` |
| `home_native` | Native | `#adHomeNative` |
| `progress_native` | Native | `#adProgressNative` |
| `session_inter` | Interstitial | `#adInter` |
| `zone_rewarded` | Rewarded | `#adReward` |

`AD_UNITS` in the prototype script is the same table with both test IDs and a
placement note per row. `app-open` is listed in the test table but deliberately
**not** used — see the rules in section 3.

## 2. Placement map

| Screen | Slot | Fires when | Why here |
| --- | --- | --- | --- |
| Home | `home_banner` | Home visible, free account, consent settled | Anchored above the tab bar, so a load never shifts content the user is reading. Highest fill, lowest cost to the user. |
| Home | `home_native` | Home visible, same gate | Sits between sections, styled in the app's own type. Feed-native, so it does not read as an interruption. |
| Progress | `progress_native` | Progress visible, same gate | Read-only screen with no interaction cost, so a static slot is free to the user. |
| After a logged craving | `session_inter` | User taps back to Home from the "It passed" screen | A genuine natural break: the task is done and logged. |
| Trigger Zones | `zone_rewarded` | Free account at the zone cap taps "Watch an ad for one more" | Opt-in, and it pays out something the user actually wants — a zone slot. |

## 3. Rules the code enforces

These come from AdMob's own implementation guidance, and they are the part worth
being strict about.

- **Paid accounts make zero ad requests.** Not "ads are hidden" — no request is
  built at all. `adsEligible()` gates on `!isPaid()`.
- **Never during a craving.** `AD_BLOCKED_VIEWS` covers `walk`, `breathe`,
  `reflect` and `cleared`, and `adBlock()` also checks `ce.iv` so nothing fires
  while a Craving Escape timer is running. A user mid-craving is the worst
  possible place for an accidental click, and it is the moment the app exists for.
- **At most one interstitial per 60 seconds.** `AD_COOLDOWN` is 60 000 ms and
  `adLast` gates `maybeInterstitial`. AdMob asks that ads persist 60s or longer
  and that a new request is not made sooner than the 60s rate when a user moves
  back and forth between screens.
- **Natural transitions only.** The interstitial fires once, after a session is
  logged. Not on app open, not on tab changes, not mid-scroll.
- **No app-open ads.** They would land on a user who just tapped "I have a
  craving", which is exactly the accidental-click case AdMob warns about.
- **Rewarded is always opt-in.** The user taps to start it and can decline.
- **Banner refresh only while visible.** Do not request when the screen is off,
  and size the anchored adaptive banner to the measured container width so the
  layout does not jump.

## 4. Consent — required before any request

Google requires a **Google-certified CMP integrated with the IAB TCF** to serve
personalised ads in:

- **EEA and UK** — since 16 January 2024
- **Switzerland** — since 31 July 2024

Without it, traffic is limited to non-personalised or limited ads. In AdMob, that
is handled by the **User Messaging Platform (UMP)** SDK plus a published European
regulations message, which is itself TCF-certified.

Three things people miss:

1. **Withdrawal must be possible at any time**, not only at first launch. GDPR
   requires it. The prototype surfaces this as **Settings → Plan & ads → Privacy
   options** (`#privacyOpen`), which re-opens the form. Map it to
   `UserMessagingPlatform.showPrivacyOptionsForm(...)`.
2. **Tag under-age-of-consent.** If your app or audience can include users under
   the age of consent, call `setTagForUnderAgeOfConsent(true)` on the request
   parameters *and* pass the `TFUA` flag on every ad request.
3. **Publish a privacy policy that names the ad partners**, and keep an
   `app-ads.txt` on your domain listing the seller IDs Google gives you.

The prototype defaults `S.consent` to `'limited'` — non-personalised only — until
the user opts in. Keeping the restrictive value as the default is deliberate.

## 5. Entitlement model

| Feature | Free | Plan |
| --- | --- | --- |
| Craving Escape | Always free, ads supported | Free, ad-free |
| Craving log, Progress | Included | Included |
| Smoke-free timer | Locked — plan only, no trial | Runs for the length of the plan |
| Trigger Zones | 1 (plus any earned by a rewarded ad) | Unlimited |
| Home & Lock Screen widgets | 3-day trial, then ad-supported | Kept, ad-free |

Constants live at the top of the prototype script: `TRIAL_MS` (the widget
trial), `ZONE_CAP_FREE`, `AD_COOLDOWN`. The helpers are `isPaid()`,
`timerUnlocked()` (paid only), `widgetsUnlocked()` (paid or in trial),
`zoneCap()` and `adsEligible()`. Enforce the same rules server-side — a
client-side paywall is a display decision, not a security boundary.

## 6. Native wiring

```kotlin
object Ads {
    // 0 = no consent yet, 1 = personalised, 2 = non-personalised
    private var consent = 0
    private var lastInter = 0L
    private var inter: InterstitialAd? = null

    private const val COOLDOWN = 60_000L
    private val BLOCKED = setOf("walk", "breathe", "reflect", "cleared")

    fun init(context: Context, onDone: (Int) -> Unit) {
        MobileAds.initialize(context) {
            UMP.requestConsentInfoUpdate(
                context,
                com.google.android.ump.ConsentRequestParameters.Builder()
                    .setTagForUnderAgeOfConsent(UNDER_AGE_OF_CONSENT)
                    .build(),
                { info, err ->
                    UMP.loadAndShowConsentFormIfRequired(context, info, {
                        consent = if (info?.consentStatus == UMP.CONSENT_STATUS_GOT) 1 else 2
                        onDone(consent)
                    }, { consent = 2; onDone(consent) })
                },
                { consent = 2; onDone(consent) })
        }
    }

    private fun eligible(): Boolean = !isSubscribed && consent != 0

    /** Banner: docked, so nothing moves when it loads. */
    fun loadBanner(adView: AdView) {
        if (!eligible()) { adView.visibility = View.GONE; return }
        adView.visibility = View.VISIBLE
        adView.adUnitId = IDS["home_banner"]!!
        val w = adView.context.resources.displayMetrics.widthMetrics.let {
            (it.widthPixels / it.density).toInt()
        }
        adView.adSize = AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
            adView.context, w)
        adView.loadAd(AdRequest.Builder().build())
        // Refresh happens automatically, and only while the view is on screen.
    }

    /** Interstitial: call only at a real transition point. */
    fun preloadInterstitial(activity: Activity) {
        if (!eligible()) return
        inter = InterstitialAd.load(activity, IDS["session_inter"], AdRequest.Builder().build()) { }
    }

    fun maybeShowInterstitial(activity: Activity, view: String) {
        if (!eligible()) return
        if (view in BLOCKED) return                 // never during a craving
        if (System.currentTimeMillis() - lastInter < COOLDOWN) return
        val ad = inter ?: return
        lastInter = System.currentTimeMillis()
        ad.show(activity)
        inter = null
        preloadInterstitial(activity)
    }

    /** Rewarded: opt-in only, and grant only from onUserEarnedReward. */
    fun offerZoneReward(activity: Activity, grant: () -> Unit) {
        if (!eligible()) return
        RewardedAd.load(activity, IDS["zone_rewarded"], AdRequest.Builder().build()) { ad ->
            ad.fullScreenContentCallback = object : FullScreenContentCallback() {
                override fun onAdDismissedFullScreenContent() = ad.destroy()
                override fun onAdFailedToShowFullScreenContent(e: AdError) = ad.destroy()
            }
            ad.show(activity) { ad.destroy(); grant() }
        }
    }

    /** UMP privacy options, re-opened from Settings. */
    fun privacyOptions(activity: Activity, onDone: () -> Unit) {
        UMP.showPrivacyOptionsForm(activity, { consent = 1; onDone() }, { onDone() })
    }
}
```

Two rules the snippets depend on that are easy to break later:

- Grant the reward in `onUserEarnedReward` only. Not on `show()`, not on dismiss.
- Destroy every ad object in its dismiss/fail callback, or you leak activity
  references.

## 7. Pre-launch checklist

- [ ] Every test ad unit ID replaced with your own
- [ ] Sample app IDs replaced in manifest / `Info.plist`
- [ ] UMP flow shipped, European regulations message **published** (not draft)
- [ ] Privacy options reachable from Settings, working after a cold restart
- [ ] TFUA flag wired if under-age-of-consent can apply
- [ ] `app-ads.txt` live on your domain
- [ ] Privacy policy names Google AdMob, the UMP SDK and the mediation partners
- [ ] `ads.txt` line present for mediation
- [ ] Entitlement rules enforced server-side, not just client-side
- [ ] Ads verified absent on walk, breathe, reflect and cleared
- [ ] Interstitial cooldown confirmed at 60s under rapid navigation
- [ ] Rewarded grant confirmed from the callback only
- [ ] Paid account confirmed to issue **zero** ad requests (watch the AdMob
      request log, not just the UI)
- [ ] Adaptive banner confirmed not to shift layout on a 360dp-wide device
- [ ] Latest Google Mobile Ads SDK, per Google's implementation guidance

## Sources

- [Implementation guidance — AdMob Help](https://support.google.com/admob/answer/2936217)
- [Banner ad guidance](https://support.google.com/admob/answer/6128877) and
  [discouraged banner implementations](https://support.google.com/admob/answer/6128877)
- [Interstitial ad guidance](https://support.google.com/admob/answer/6066980)
- [Enable test ads (Android)](https://developers.google.com/admob/android/test-ads)
- [Test ad units (Android)](https://developers.google.com/admob/android/next-gen/ad-inspector/test-ad-units)
- [Banner setup (Android)](https://developers.google.com/admob/android/banner)
- [Interstitial setup (Android)](https://developers.google.com/admob/android/next-gen/interstitial)
- [Rewarded interstitial setup (Android)](https://developers.google.com/admob/android/next-gen/rewarded-interstitial)
- [App open ads (Android)](https://developers.google.com/admob/android/app-open)
- [Native ads (Android)](https://developers.google.com/admob/android/next-gen/native)
- [Disclose to EEA users / GDPR (Android)](https://developers.google.com/admob/android/privacy/gdpr)
- [Google consent requirements for publishers](https://support.google.com/admob/answer/13554116)
