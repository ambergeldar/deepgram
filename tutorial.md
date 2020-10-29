# Transcribe a Recording

Deepgram uses state-of-the-art speech recognition technology and cutting-edge model training to live transcribe your recordings. Our SpeechEngine API gives you streamlined access to automatic transcription for audio in nearly every format available.

In this guide, you will learn how to use Python with our SpeechEngine API to retrieve a transcript of an audio recording. To make things easier, we'll use a sample file we've already uploaded, but you can also use your own file.

To learn more about our model training and data-labeling, visit [MissingControl API Docs](https://missioncontrol.deepgram.com/docs).

## Before you begin

Before you begin this guide you’ll need:

* Basic Python knowledge and a local development environment for Python 3. You should be comfortable with Python’s syntax, structure, and some built-in functions.
* Flask. In this tutorial, we use Flask, a Python microframework, to develop our web application. [Learn how to install Flask](https://flask.palletsprojects.com/en/1.1.x/installation/).
::: warning 
Flask uses a simple web server to serve applications in a development environment. This means that the Flask debugger is running to make catching errors easier. Do not use the development server in a production deployment. To learn more about deploying to production, see Flask's [Deployment Options](https://flask.palletsprojects.com/en/1.1.x/deploying/) documentation.
:::
* Your Deepgram username and password. If you don't already have these, [sign up for a Deegram account](https://enterprise.deepgram.com/).

## Steps

1. [Set up base application](#set-up-base-application): Set up a basic application.
2. [Configure parameters](#configure-parameters): Configure the parameters you want to send to Deepgram.
3. [Call SpeechEngine API](#call-speechengine-api): Call the SpeechEngine API and receive the response.
4. [Build transcript](#build-transcript): Build an audio transcript.

### Set up base application

First, we will set up a basic Flask application that has routes. Create a Python file called  `transcribe_audio.py`:

```
from flask import Flask, render_template
app = Flask(__name__)

@app.route("/")
def get_transcript():
	return render_template('index.html')
```

Let’s break down what’s happening here.

1. We import our Flask and render_template dependency
2. We create an instance of a Flask application, passing in the name of the application.
3. We set up a [route](/other/def.md) that executes the `get_transcript()` function when the route is visited. The function must return a string or a rendered template, and in this case, we return the `index.html` template.

Now, let's set up our `index.html` template, which is where the user will be redirected when the app is run:

```
<!doctype html>
<html>
    <head>
        <title>SpeechEngine API Audio Transcripts</title>
    </head>
    <body>
        <h1>Audio Transcripts</h1>
    </body>
</html>
```

Run the app in the command line using the following command: 

```
FLASK_APP=deepgram.py flask run 
```

You should see something like `Running on http://localhost:5000/` in your terminal. And if you visit the link in your browser, you'll see a page that says "Audio Transcripts".

### Configure parameters

Now we need to configure the parameters we for the SpeechEngine API.

First, let's set the URL for the Deepgram SpeechEngine API `/listen` endpoint, which is where we will send our HTTP request:

```
...
app = Flask(__name__)

# set Deepgram API URL
api_url = 'https://brain.deepgram.com/v2/listen'

...
```

To authenticate, we'll use Basic Authentication (over HTTPS), using our Deepgram username and password.

```
...
# set Authorization header
username = 'YOUR_USERNAME'
password = 'YOUR_PASSWORD'
headers = { 'Authorization': 'Basic {0}'.format(base64.b64encode('{}:{}'.format(username, password).encode('utf-8')).decode('utf-8')) }
...
```

Then we will pass in the URL for the audio file that we want to transcribe. For this example, we'll use the sample file we've already uploaded. However, if you have an audio file you would prefer to use instead, you can post its binary contents instead.

::: note
Deepgram supports over 40 audio formats, including the popular WAV, MP3, M4A, FLAC, and Opus. For a full list of supported formats, see [Supported Audio Formats](/other/new-doc.md).
:::

```
...
# set audio_file
audio_file = 'https://static.deepgram.com/examples/interview_speech-analytics.wav'
...
```

Finally, we will set the parameters we want to pass to the API. Deepgram will use the parameters we pass to customize its response.

```
...
# set chosen parameters
params = {
    'model': 'phonecall',
    #'model': 'meeting',
    #'model': {version_id},
    #'language': 'es',
    #'punctuate': 'true',
    #'alternatives': '3',
    'keywords': ['KPI','chew']
}
...
```

#### Parameters

In this example, we use the following parameters. To explore all of the parameters the SpeechEngine API accepts, see [SpeechEngine API Reference](https://docs.deepgram.com/).

| Name            | Default | Description |
|-----------------|---------|-------------|
| `model`    | `general` | AI model used to process uploaded audio. Standard Deepgram models include:<ul><li>general: a good, general-purpose model for everyday audio processing. If you aren't sure what model to select, start with this one.</li><li>phonecall: optimized for low-bandwidth audio phone calls.</li><li>meeting: optimized for conference room settings: multiple speakers with a single microphone.</li></ul>You may also use a custom model associated with your account by including its `version_id`.<br><br>Can be set to multiple values when the `multichannel` parameter is set to `true`. In this case, you can apply different models to separate audio channels (e.g., set `model` to `general:phonecall`, which applies the `general` model to channel 0 and the `phonecall` model to channel 1). |
| `language` | `en-US` | `BCP-47` language tag that hints at the primary spoken language. Language support is currently optimized for the following languages:<ul><li>English (<code>en-US</code>, <code>en-GB</code>, <code>en-NZ</code>)</li><li>Spanish (<code>es</code>)</li><li>Korean (<code>ko</code>)</li><li>French (<code>fr</code>)</li><li>Portuguese (<code>pt</code>, <code>pt-BR/code>)</li><li>Russian (<code>ru</code>)</li></ul>If a requested model and language combination doesn't exist, the request will fail.(?)<br><br>Note to self: API docs say that the language will be guessed. But there's a default value if no language is provided. Does it guess, or use the default? |
| `diarize`     | `false` | Indicates whether to recognize speaker changes. If `true`, each word in the transcript will be assigned a speaker number starting at 0. |
| `punctuate` | `false` | Indicates whether to add punctuation to the transcript. |
| `multichannel` | `false` | Indicates whether to isolate speakers on independent audio channels. When set to `true`, you will receive one transcript for each channel, which means you can apply a different model to each channel using the `model` parameter (e.g., set `model` to `general:phonecall`, which applies the `general` model to channel 0 and the `phonecall` model to channel 1).<br><br>Must be set to `true` when the `uttseg` parameter is set to `true`. |
| `uttseg` | `false` | Indicates whether to segment speech into meaningful semantic units, which allows the model to interact more naturally and effectively with speakers' spontaneous speech patterns. For example, when humans speak to each other conversationally, they often pause mid-sentence to reformulate their thoughts, or stop and restart a badly-worded sentence. When `uttseg` is set to `true`, these utterance segments are identified, so you can present the transcript with a more natural speaking pattern.<br><br>When set to `true`, the `multichannel` parameter must also be set to `true`. |

### Call SpeechEngine API

Now that we have configured our desired parameters, we will call the API and pass the response to `index.html`. To do so, modify `transcribe_audio.py` as follows:

```
...
@app.route("/")
def get_transcript():

    # Call Deepgram API and get response
    response = requests.post(api_url, data, headers=headers, params=params)

    # if successful, load returned JSON object to body parameter, and pass to `index.html`
    if response.status_code == 200:
        encodedJSON=json.dumps(response.json())
        body = json.loads(encodedJSON)
        return render_template('index.html', body=body)

    # else, return an error message
    else:
        return render_template('index.html', error='No data returned')

...
```

To handle the parameters being passed, we'll also modify `index.html`:

```
...
    <body>
        <h1>Audio Transcripts</h1>

        {% if body is not defined%}
            {% if error is defined %}
                An error occurred.
            {% endif %}
        {% else %}
            {{ body }}
        {% endif %}
     </body>
...
```

When you refresh your browser, you should see the response object in [JSON format](/other/def.md).

### Build transcript

Now that we have the response from the server, let's add some formatting to build a more useful display in our browser.

Beneath our `get_transcript` function, let's add a function to display a header:

```
def build_header(data, audio_file):

    # Build header object
    header = {
        'Created': '{}'.format(data['metadata']['created']),
        'Duration': '{:.2f} s'.format(data['metadata']['duration']),
        'Channels': '{}'.format(data['metadata']['channels']),
        'File': '{}'.format(audio_file)
    }

    return header
```

And beneath that, let's add a function to feed the alternative transcriptions into the body:

```
def build_body(data):
    body={}

    #get alternatives
    alternatives = data['results']['channels'][0]['alternatives']
    
    body=alternatives

    return body

```

Now, let's modify our `get_transcript` function to call these sub-functions:

```
# Call Deepgram API
    response = requests.post(api_url, data, headers=headers, params=params)

    if response.status_code == 200:
        encodedJSON=json.dumps(response.json())
        header = build_header(json.loads(encodedJSON), audio_file)
        body = build_body(json.loads(encodedJSON))
        return render_template('index.html', header=header, body=body)
```

Finally, to handle these new functions, we'll modify `index.html`. To handle the new `header` and `body`, we'll add:

```
...
    <body>
        <h1>Audio Transcripts</h1>

        {% if header is not defined or body is not defined%}
            {% if error is defined %}
                An error occurred.
            {% endif %}
        {% else %}
        <p>
            Transcript words are color-coded based on confidence level.<br><br>
            High: Greater than 90%<br>
            <span class="confidence-mid">Medium: Between 90% and 50%</span><br>
            <span class="confidence-low">Low: Under 50%</span>
        </p>

        {% for key, value in header.items() %}
            {{key}}:
            {{value}}<br>
        {% endfor %}
        <br><br>

        {# for each alternative #}
        {% for alternative in body %}

            {# output the alternative number and overall confidence level #}
            <span class="alternative">Alternative {{ loop.index }}</span><br>
            <span class="confidence">(confidence: {{ "%.2f%%"|format(body[0]['confidence']*100) }})</span>
            <p>
            {% set transcript = body[0]['transcript'].split(' ') %}

            {# for each word in the transcript #}
            {% for word in transcript%}

                {# if the first word in the transcript #}
                {% if loop.previtem is not defined %}

                    {# if speaker exists, print speaker #}
                    {% if body[0]['words'][loop.index0]['speaker'] is defined %}
                        Speaker {{ body[0]['words'][loop.index0]['speaker']+1 }}:&nbsp;
                    {% endif %}

                    {# print timestamp #}
                    [{{ format_transcript_time(body[0]['words'][loop.index0]['start']) }}]

                {# if any other word in transcript #}
                {% else %}

                    {# if speaker exists #}
                    {% if body[0]['words'][loop.index0]['speaker'] is defined %}

                        {# if new speaker, print Speaker label and timestamp #}
                        {% if body[0]['words'][loop.index0]['speaker'] != body[0]['words'][loop.index0-1]['speaker']  %}
                            <br><br>Speaker {{ body[0]['words'][loop.index0]['speaker']+1 }}:&nbsp;
                            [{{ format_transcript_time(body[0]['words'][loop.index0]['start']) }}]

                        {# else if sentence ends with punctuation, print timestamp #}
                        {% elif loop.previtem.endswith(('.', '?', '!')) %}
                            [{{ format_transcript_time(body[0]['words'][loop.index0]['start']) }}]
                        {% endif %}
                    
                    {# else if sentence ends with punctuation, print timestamp #}
                    {% elif loop.previtem.endswith(('.', '?', '!')) %}
                        [{{ format_transcript_time(body[0]['words'][loop.index0]['start']) }}]
                    {% endif %}
                {% endif %}

                {# color-code words by confidence level #}
                {% if body[0]['words'][loop.index0]['confidence'] < 0.50 %}
                    <span class=confidence-low>
                {% elif 0.50 <= body[0]['words'][loop.index0]['confidence'] < 0.90  %}
                    <span class=confidence-mid>
                {% endif %}
                {{ word }}
                </span>

            {% endfor %} 
            </p>

        {% endfor %}
    
        {% endif%}
     </body>
...
```

When you refresh your browser, you should see the following:

![Sample Audio Transcript](/images/sample-audio-transcript.png)

## Keep reading

* [Improving model performance by labeling data and training](https://missioncontrol.deepgram.com/docs)
