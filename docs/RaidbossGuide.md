# Triggers File Format

## File Structure

```javascript
[{
  zoneRegex: /match for the zone/,
  timelineFile: 'filename.txt',
  timeline: `hideall`,
  timelineReplace: [
  {
     locale: 'en',
     replaceText: {
      'regexSearch': 'strReplace',
     },
     replaceSync: {
      'regexSearch': 'strReplace',
     },
   },
  ],
  resetWhenOutOfCombat: true,
  triggers: [
    { /* ..trigger 1.. */ },
    { /* ..trigger 2.. */ },
    { /* ..trigger 3.. */ },
  ]
},
{
  zoneRegex: /match for another zone/,
  triggers: [
    { /* ..trigger 1.. */ },
    { /* ..trigger 2.. */ },
    { /* ..trigger 3.. */ },
  ]
}]
```

### Elements

**zoneRegex**
Largely self-explanatory. For most situations you will want to match the encounter zone as reported by ACT and nothing else.

**timelineFile**
An optional timeline file to load for this zone. These files live alongside their parent trigger file in the appropriate folder. (As for example `raidboss/data/04-sb/raid/`).

**timeline**
Optional extra lines to include as part of the timeline.

**locale**
Optional locale to restrict the trigger file to, e.g. 'en', 'ko', 'fr'. If not present, applies to all locales.

**replaceText**
Key:value pairs to search and replace in timeline ability names. The display name for that ability is changed, but all `hideall`, `infotext`, `alerttext`, `alarmtext`, etc all refer to the original name. This enables translation/localization of the timeline files without having to edit those files directly.

**replaceSync**
Key:value pairs to search and replace in timeline file sync expressions. Necessary if localized names differ in the sync regexes.

**resetWhenOutOfCombat**
Boolean, defaults to true. If true, timelines and triggers will reset automatically when the game is out of combat. Otherwise it's necessary to manually call `data.StopCombat()`.

## Trigger Structure

```javascript
{
  id: 'id string',
  disabled: false,
  regex: /trigger-regex-(with-position-1)(and-position-2)-here/,
  regexFr: /trigger-regex-(with-position-1)-here-but-in-french/,
  condition: function(data, matches) { return true if it should run },
  preRun: function(data, matches) { do stuff.. },
  delaySeconds: 0,
  durationSeconds: 3,
  suppressSeconds: 0,
  promise: function(data, matches) { return promise to wait for resolution of },
  sound: '',
  soundVolume: 1,
  response: {
    alarmText: {'Alarm Popup'},
    alertText: {'Alert Popup'},
    infoText: {'Info Popup'},
    tts: {'TTS text'},
  },
  alarmText: {en: 'Alarm Popup'},
  alertText: {en: 'Alert Popup'},
  infoText: {en: 'Info Popup'},
  tts: {'TTS text'},
  run: function(data, matches) { do stuff.. },
},
```

### Trigger Elements

**id string**
 An id string for the trigger, used to disable triggers. Every built-in trigger that has a text/sound output should have an id so it can be disabled. User-defined triggers need not have one.

 **disabled: false**
If this is true, the trigger is completely disabled and ignored.
Defaults to false.

 **regex**
The regular expression Cactbot will run against each log line to determine whether the trigger will activate. Positions 1 and 2 are capture groups that are returned in the `matches` variable for use in trigger functions.

More commonly, however, a regex replacement is used instead of a bare regex.
Helper functions defined in [regexes.js](https://github.com/quisquous/cactbot/blob/master/resources/regexes.js)
take the parameters that would otherwise be extracted via match groups.
From here, the functions automagically construct the regex that should
be matched against.

**regexFr**
Example of a locale-based regular expression for the 'fr' locale.
If `Options.Language == 'fr'`, then `regexFr` (if it exists) takes precedence over `regex`.
Otherwise, it is ignored.  This is only an example for french, but other locales behave the same, e.g. regexEn, regexKo.
(Locale regexes do not have a defined ordering.
Current practice is to order them as `de`, `fr`, `ja`, `cn`, `ko`, however.
Additionally, as with bare `regex` elements, current practice is to use regex replacements instead.)

**condition: function(data, matches)**
Activates the trigger if the function returns `true`. If it does not return `true`, nothing is shown/sounded/run. If multiple functions are present on the trigger, this has first priority to run.
(Pre-made "canned" conditions are available within [conditions.js](https://github.com/quisquous/cactbot/blob/master/resources/conditions.js).
Generally speaking it's best to use one of these if it fits the situation.)

**preRun: function(data, matches)**
If the trigger activates, the function will run as the first action after the activation condition is met.

**promise: function(data, matches)**
If present and a function which returns a promise, will wait for promise to resolve before continuing with trigger. This runs after `preRun` and before the `delaySeconds` delay.

**delaySeconds**
An amount of time, in seconds, to wait from the time the regex match is detected until the trigger activates. May be a number or a function(data, matches) that returns a number.

**durationSeconds**
Time, in seconds, to display the trigger text. May be a number or a `function(data, matches)` that returns a number. If not specified, defaults to 3.

**suppressSeconds**
Time to wait, in seconds, before showing this trigger again.  May be a number or a function(data, matches).  The time to wait begins at the time of the initial regex match, and is unaffected by presense or absence of a delaySeconds value. Once a trigger with this element activates, it will not activate again until after its timeout period is over.

**sound**
Sound file to play, or one of 'Info', 'Alert', 'Alarm', or 'Long'. Paths to sound files are relative to the ui/raidboss/ directory.

**soundVolume**
Volume between 0 and 1 to play the sound associated with the trigger.

**response**
A way to return infoText/alertText/alarmText/tts all from a single entrypoint.
Also used by `resources/responses.js`.
Response has less priority than an explicity specified text or tts,
and so can be overridden.
(As with `regex` and `condition`, "canned" responses are available within [responses.js](https://github.com/quisquous/cactbot/blob/master/resources/responses.js).)

**alarmText**
Displays a text popup with Alarm importance when the trigger activates. This is for high-priority events where failure is guaranteed to kill you, is likely to wipe the encounter, or will otherwise make successful completion much more difficult. (Examples include Allagan Rot in T2, Cursed Shriek in T7, or Ultros' Stoneskin cast in O7s. ) May be a string or a `function(data, matches)` that returns a string.

**alertText**
Displays a text popup with Alert importance when the trigger activates. This is for medium-priority events that might kill you, or inflict party-wide damage/debuffs (For example, warning the main tank that a buster is incoming, or warning the entire party of an upcoming knockback.) May be a string or a `function(data, matches)` that returns a string.

**infoText**
Displays a text popup with Info importance when the trigger activates. This is for low-priority events that will be merely annoying if not attended to immediately. (For example, warning of an add spawn, or informing healers of incoming raid damage.) May be a string or a `function(data, matches)` that returns a string.

**tts**
An alternative text string for the chosen TTS option to use for callouts. This can be a localized object just like the text popups.

**run: function(data, matches)**
If the trigger activates, the function will run as the last action before the trigger ends.

## Miscellaneous Trigger Info

Any field that can return a function (e.g. `infoText`, `alertText`, `alarmText`, `tts`) can also return a localized
object, e.g. instead of returning 'Get Out', they can return {en: 'Get Out', fr: 'something french'}
instead.  Fields can also return a function that return a localized object as well.  If the current locale
does not exist in the object, the 'en' result will be returned.

Trigger elements are evaluated in this order, and must be listed in this order:

- id
- disabled
- regex (and regexDe, regexFr, etc)
- beforeSeconds (for timelineTriggers)
- condition
- preRun
- delaySeconds
- durationSeconds
- suppressSeconds
- (the delaySeconds occurs here)
- promise
- (awaiting the promise occurs here)
- sound
- soundVolume
- response
- alarmText
- infoText
- groupTTS
- tts
- run

## Timeline Info

The trigger subfolders may contain timeline text files in the format defined by ACT Timeline plugin, which described in here:
<http://dtguilds.enjin.com/forum/m/37032836/viewthread/26353492-act-timeline-plugin>

Each timeline file Cactbot uses has to be loaded by a relative directory reference from the given [TRIGGER-FILE].js. Typically the filename for the timeline file will match the name of the trigger file, and for specific encounters the filenames should at least loosely match the zone name.

Cactbot implements some extensions to the original format. These extensions can appear in the file
itself or in the `timeline` field in the triggers:

**infotext "event name" before 1**
Show a info-priority text popup on screen before an event will occur. The `event name` matches a timed event in the file and will be shown before each occurance of events with that name. By default the name of the event will be shown, but you may specify the text to be shown at the end of the line if it should be different. The `before` parameter must be present, but can be 0 if the text should be shown at the same time the event happens. Negative values can be used to show the text after the event.

**Example infotext which shows the event name 1s before the event happens**
`infotext "event name" before 1`

**Example infotext which specifies different text to be shown earlier**
`infotext "event name" before 2.3 "alternate text"`

**Example alert-priority popups using the same parameters**
`alerttext "event name" before 1`
`alerttext "event name" before 2.3 "alternate text"`

**Example alarm-priority popups using the same parameters**
`alarmtext "event name" before 1`
`alarmtext "event name" before 2.3 "alternate text"`