# Rasa for Botfront

**Rasa for Botfront** is a maintained forked version of [Rasa](https://github.com/rasaHQ/rasa) to be used with [Botfront project on Github](https://github.com/botfront/botfront), an enterprise-grade open source conversational platform built for Rasa teams.

For more information visit the [Botfront project on Github](https://github.com/botfront/botfront)

## Main features supported by this fork (when used with Botfront)
- Multilingual support (incl. NLG)
- Form UI support
- Conversion routes
- Fetching creds and endpoints at startup
- support for gazette (that is not really important)
- Automated tests
- NLU evaluation


## Integrating Rasa upstream changes

1. If you don't have the rasa upstrem do
```bash
git remote add upstream https://github.com/RasaHQ/rasa.git
```
2. Then retreive all the tags
```bash
 git fetch upstream --tags 
```
3. Merge the most recent one
```bash
 git merge TAG
```
4. Solve conflicts
5. Apply versions using
```bash
 command to release here
```

## Rasa Addons

Rasa for Botfront is packaged with custom componenents used by Botfront.

### `rasa_addons.core.policies.BotfrontDisambiguationPolicy`

This policy implements fallback and suggestion-based disambiguation.

It works with actions ``rasa_addons.core.actions.ActionBotfrontDisambiguation``, ``rasa_addons.core.actions.ActionBotfrontDisambiguationFollowup`` and ``rasa_addons.core.actions.ActionBotfrontFallback``, and NLU pipeline component ``rasa_addons.nlu.components.intent_ranking_canonical_example_injector.IntentRankingCanonicalExampleInjector``.

#### Example usage

```yaml
policies:
  ...
  - name: rasa_addons.core.policies.BotfrontDisambiguationPolicy
    fallback_trigger: 0.30 # default value
    disambiguation_trigger: '$0 < 2 * $1' # default value
    disambiguation_template: 'utter_disambiguation' # default value
    n_suggestions: 2 # default value
    excluded_intents:
      - ^chitchat\..* # default value
      - ^basics\..*
  ...
```

#### Note: Automatic generation of suggestion button titles

Botfront introduces the notion of "canonical" training examples, which provide a canonical human-readable text for intent labels. For example, for an intent ``pay_bills`` with examples "Pay bills", "I want to pay my bills", "How does one pay the bills on this website?", the first example may be selected as canonical. Canonical status serves as a cue to the bot designer, since intent labels can become untractable over time. It may also come to serve more Botfront-internal roles in the future.

The Botfront Disambiguation Policy uses canonical status to provide localized text for the suggestion buttons shown to users during disambiguation. In order to enable this feature, the NLU pipeline for each language model needs to be extended in the following way:

```yaml
pipeline:
  ...
  - name: rasa_addons.nlu.components.intent_ranking_canonical_example_injector.IntentRankingCanonicalExampleInjector
  ...
```

This NLU component enriches the ``intent_ranking`` key of user messages in the tracker with canonical text, so that the Disambiguation Policy may pick it up. If the NLU component is not used, buttons will have the intent name as their title.

#### Parameters

##### fallback_trigger

Float (default ``0.30``): if confidence of top-ranking intent is below this threshold, fallback is triggered. Fallback is an action that utters the template ``utter_fallback`` and returns to the previous conversation state.

##### disambiguation_trigger

String (default ``'$0 < 2 * $1'``): if this expression holds, disambiguation is triggered. (If it has already been triggered on the previous turn, fallback is triggered instead.) Here this expression resolves to "the score of the top-ranking intent is below twice the score of the second-ranking intent". Disambiguation is an action that lets the user to choose from the top-ranking intents using a button prompt.

##### disambiguation_template

String (default ``'utter_disambiguation'``): a response name resolving to a template containing a ``text`` field with a message, e.g. "I could not quite understand. Did you mean...". Any button included under the ``buttons`` field of this template will also appear at the end of autogenerated suggestions, e.g. ``{"title": "None of the above", "type": "postback", "payload": "/deny_suggestions"}``.

##### n_suggestions

Int (default ``2``): the maximum number of suggestions to display (excluding the 'Other' options).

##### excluded_intents

List (default ``["^chitchat\..*", "^basics\..*"]``): any intent (exactly) matching one of these regular expressions will not be shown as a suggestion.


### `rasa_addons.core.channels.webchat.WebchatInput`

#### Example usage

```yaml
credentials:
  ...
  rasa_addons.core.channels.webchat.WebchatInput:
    session_persistence: true
    base_url: {{rasa_url}}
    socket_path: '/socket.io/'
  ...
```

### `rasa_addons.core.channels.rest.BotfrontRestInput`

Rest Input Channel with multilanguage and metadata support.

#### Example usage

```yaml
credentials:
  ...
  rasa_addons.core.channels.rest.BotfrontRestInput:
    # POST {{rasa_url}}/webhooks/rest/webhook/
  ...
```

### `rasa_addons.core.channels.bot_regression_test.BotRegressionTestInput`:
Conversation testing channel. Simulates each user event in the input test stories as a message sent by a user, then compares the input story to the results from rasa. Returns a diff of the input story and output story with expected and actual events.

#### Example usage

```yaml
credentials:
  ...
  rasa_addons.core.channels.bot_regression_test.BotRegressionTestInput: {}
    # POST {{rasa_url}} /webhooks/bot_regression_test/run
  ...
```

### `rasa_addons.core.nlg.BotfrontTemplatedNaturalLanguageGenerator`

Idential to Rasa's `TemplatedNaturalLanguageGenerator`, except in handles templates with a language key.

### `rasa_addons.core.nlg.GraphQLNaturalLanguageGenerator`

The standard way to connect to the Botfront NLG endpoint.

#### Example usage

```yaml
endpoints:
  ...
  nlg:
    url: 'http://localhost:3000/graphql'
    type: 'rasa_addons.core.nlg.GraphQLNaturalLanguageGenerator'
  ...
```