
<style
  type="text/css">
style {color:#ffffff;display:hidden}
h1, h2, h3, h4, h5, h6 {color:#333333;}
p, li {color:#333333}
code {color:#000080;}
</style>

https://workshop.simsimi.com/document

# Daily Conversation API
SimSimi Workshop Services (SWS) provides services based on the daily conversation chatbot 'SimSimi', which has provided enjoyment to approximately 350 million users worldwide. The reason why 'SimSimi' has been able to provide continuous and successful services is because more than 22 million panelists from all over the world with diverse backgrounds have created vivid and inspiring content with their own wit and inspiration.

## How it works
Each talkset consists of a pair of question sentences (`qtext`) and answer sentences (`atext`).

<img src="https://workshop.simsimi.com/static/img/smalltalk_diagram_01.png" width="300px" alt="대화세트 개념도">

The SimSimi Dialogue Processing Engine (AICR) searches for appropriate responses in the dialogue set repository, which contains numerous dialogue sets. When a request is received through the daily conversation API, AICR searches the dialogue set repository for question sentences (`qtext`) with high similarity to the user sentence (`utext`) and creates a candidate group, and considers the parameters included in the request and other conditions to find the most appropriate response.

The response sentence(`atext`) provided by the daily conversation API is the response sentence(`atext`) of the conversation set selected in this process. For example, when the `utext` of the request is "Have you eaten?", the daily conversation API returns "Yes, I ate" as `atext` through the following process.

<img src="https://workshop.simsimi.com/static/img/smalltalk_diagram_02.png" width="600px" alt="일상대화 API 개념도">

## 기본요청
You can receive a response by making a POST request to the Daily Conversation API endpoint (`https://wsapi.simsimi.com/{VERSION}/talk`) specifying the project key and two required parameters (user sentence `utext`, language code `lang`). 

#### Example request
``` bash
curl -X POST https://wsapi.simsimi.com/190410/talk \
     -H "Content-Type: application/json" \
     -H "x-api-key: PASTE_YOUR_PROJECT_KEY_HERE" \
     -d '{
            "utext": "Hi. There", 
            "lang": "en" 
     }'                     
```
- `utext` : User sentences
- `lang` : User's language code ([Languages and language codes supported by the Daily Conversation API](#ExampleResponse)

#### Example response
``` json
{
  "status":200,
  "statusMessage":"Ok",
  "atext":"뭐이눔아",
  "lang":"ko",
  "request":{
    "utext":"Hi. There",
    "lang":"en"
    }
}    
```
- `atext` : Answer sentence 
- `request` : Request body
- `status`, `statusMessage` : Status information (See [Status code table](#StatusCodeTable))

## Response control
We provide options to adjust responses to provide responses tailored to the characteristics of each chatbot.

#### Example request
If you want to receive only sentences with a [Bad word probability](#BadWordProbability) of 70% or less from among the conversation sets generated in Korea or the United States as answers, you can make a request by adding the two options `country` and `atext_bad_prob_max` as follows.
``` bash
curl -X POST https://wsapi.simsimi.com/190410/talk \
     -H "Content-Type: application/json" \
     -H "x-api-key: PASTE_YOUR_PROJECT_KEY_HERE" \
     -d '{
            "utext":"Hi. There",
            "lang": "en",
            "country" : ["KR", "US"],
            "atext_bad_prob_max": 0.7
     }'  
 ```
#### Response Control Options

- `country` : Country filter for conversation sets. For languages that are spoken in multiple countries, such as English or Spanish, you can limit your response candidates to conversation sets created in a specific countries. ([ISO-3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements) You can list up to 10 country codes, if not specified, all countries are targeted.)  
　
- `atext_bad_prob_max`, `qtext_bad_prob_max`, `talkset_bad_prob_max` : Maximum probability of bad words in a sentence. It is used to suppress bad language in chatbot responses. In many cases, it is sufficient to appropriately adjust the maximum bad word probability of the response sentence (`atext_bad_prob_max`), and the bad word probability of the question sentence and the dialogue set. (`qtext_bad_prob_max`) (`talkset_bad_prob_max`)You can control more conservatively by specifying additionally. The probability of bad words in the dialogue set is determined by combining the question sentence and the answer sentence as a single sentence. The probability value with one decimal place (0.0 ~ 1.0, default value 1.0 if not specified) 
　
- `atext_bad_prob_min` : Minimum probability of bad words in the response sentence. This option can be used when the chatbot mainly uses bad words. Many chatbot platforms have restrictions related to the soundness of the content, so please use it with caution. Probability value with one decimal place (0.0 ~ 1.0, default value 0.0 if not specified)  
　
- `atext_length_max`, `atext_length_min` : Specify the length range of the response sentence. You can set the length range of the response sentence according to the nature of the chatbot or the conversation situation. (Integer from 1 to 256, if not specified, the default value `atext_length_max` is 256, `atext_length_min` is 1)  
　
- `regist_date_max`, `regist_date_min` : Specify the registration date range of the talkset. You can use it to implement chatbots that are sensitive to the latest trends, chatbots that are stuck in the past, etc. (Use in the format `yyyy-MM-dd HH:mm:ss`, if not specified, the default value `regist_date_max` is the current time, `regist_date_min` is the first talkset registration date)
　  


## Additional information 
The daily conversation API provides a way to get more information about a response. You request additional information by listing it in the `cf_info` object in the request body, as in the example below.
#### Example request
``` bash
curl -X POST https://wsapi.simsimi.com/190410/talk \
     -H "Content-Type: application/json" \
     -H "x-api-key: PASTE_YOUR_PROJECT_KEY_HERE" \
     -d '{
            "utext":"Hi. There",
            "lang": "en",
            "cf_info" : [
                  "qtext",
                  "country",
                  "atext_bad_prob",
                  "atext_bad_type",
                  "regist_date"
            ]
     }'         
```
#### Example response
``` json
{
    "status" : 200,
    "statusMessage" : "OK",
    "atext" : "Hello, is the weather nice today?",
    "lang" : "en",
    "utext" : "Hi. There",
    "qtext" : "Hi~!",
    "country" : "KR",
    "atext_bad_prob" : 0.0,
    "atext_bad_type" : "dpd",
    "regist_date" : "2017-07-08 08:24:37"
}                           
  
```
#### Request additional information options
- `qtext` : Question sentence (`qtext`Question sentence (`qtext`) paired with answer sentence (`atext`)
- `country`: Country code of the country where the dialogue set is created - `atext_bad_prob` :  Probability of bad words in the answer sentence (`atext`)
- `atext_bad_type`: Probability of bad words in the answer sentence (`atext_bad_prob`) The discrimination method used in the estimation. One of `STAPX`, `DPD`, `WPF`, or `HB10A`. ([Detailed description of the discrimination method](http://blog.simsimi.com/2019/03/blog-post.html) )
- `regist_date` : When the conversation set is created


## Supported Languages
Most language codes are identical to ISO-639-1, but note that there are some exceptions (*).

|Language name	|	 Language name (native)	|	 Language code|
| --- | --- | --- |
|Galician	|	 galego	|	 gl|
|Gujarati	|	 ગુજરાતી	|	 gu|
|Georgian	|	 ქართული	|	 ka|
|Greek	|	 Ελληνικά	|	 el|
|Dutch	|	 Nederlands	|	 nl|
|Nepali	|	 नेपाली	|	 ne|
|Norwegian (Bokmål)	|	 norsk	|	 nb|
|Danish	|	 Dansk	|	 da|
|German	|	 Deutsch	|	 de|
|Latvian	|	 latviešu	|	 lv|
|Russian	|	 русский	|	 ru|
|Romanian	|	 Română	|	 ro|
|Lithuanian	|	 lietuvių	|	 lt|
|Marathi	|	 मराठी	|	 mr|
|Macedonian	|	 македонски	|	 mk|
|Malayalam	|	 മലയാളം	|	 ml|
|Malay	|	 Melayu	|	 ms|
|Mongolian	|	 Монгол	|	 mn|
|Basque	|	 euskara	|	 eu|
|Burmese	|	 မြန်မာဘာသာ	|	 my|
|Vietnamese	|	 Tiếng Việt	|	 vn*|
|Belarusian	|	 беларуская	|	 be|
|Bengali	|	 বাংলা	|	 bn|
|Bosnian	|	 bosanski	|	 bs|
|Bulgarian	|	 български	|	 bg|
|Serbian	|	 српски	|	 rs*|
|Cebuano	|	 Cebuano	|	 cx*|
|Swahili	|	 Kiswahili	|	 sw|
|Swedish	|	 svenska	|	 sv|
|Spanish	|	 español	|	 es|
|Slovak	|	 slovenčina	|	 sk|
|Slovenian	|	 slovenščina	|	 sl|
|Sinhalese	|	 සිංහල	|	 si|
|Arabic	|	 العربية	|	 ar|
|Armenian	|	 հայերեն	|	 hy|
|Icelandic	|	 íslenska	|	 is|
|Azerbaijani	|	 Azərbaycanca	|	 az|
|Afrikaans	|	 Afrikaans	|	 af|
|Albanian	|	 Shqip	|	 al*|
|Estonian	|	 eesti	|	 et|
|English	|	 English	|	 en|
|Urdu	|	 Урду	|	 ur|
|Uzbek	|	 O‘zbek	|	 uz|
|Ukrainian	|	 українська	|	 uk|
|Welsh	|	 Cymraeg	|	 cy|
|Italian	|	 Italiano	|	 it|
|Indonesian	|	 Bahasa Indonesia	|	 id|
|Japanese	|	 日本語	|	 ja|
|Chinese	|	 中文	|	 ch*|
|Czech	|	 čeština	|	 cs|
|Kazakh	|	 қазақ	|	 kk|
|Catalan	|	 català	|	 ca|
|Kannada	|	 ಕನ್ನಡ	|	 kn|
|Cambodian	|	 ភាសាខ្មែរ	|	 kh*|
|Kurdish	|	 Kurdî (Kurmancî)	|	 ku|
|Croatian	|	 hrvatski	|	 hr|
|Tagalog	|	 Filipino	|	 ph*|
|Tamil	|	 தமிழ்	|	 ta|
|Tajik	|	 Тоҷикӣ	|	 tg|
|Thai	|	 ภาษาไทย	|	 th|
|Turkish	|	 Türkçe	|	 tr|
|Telugu	|	 తెలుగు	|	 te|
|Pashto	|	 پښتو	|	 ps|
|Punjabi	|	 ਪੰਜਾਬੀ	|	 pa|
|Persian	|	 فارسی	|	 fa|
|Portuguese	|	 português	|	 pt|
|Polish	|	 polski	|	 pl|
|French	|	 Français	|	 fr|
|Frisian	|	 Frysk	|	 fy|
|Finnish	|	 suomi	|	 fi|
|Korean	|	 한국어	|	 ko|
|Hungarian	|	 magyar	|	 hu|
|Hebrew	|	 עברית	|	 he|
|Hindi	|	 हिन्दी	|	 hi|
|Assamese	|	 অসমীয়া	|	 as|
|Breton	|	 Brezhoneg	|	 br|
|Guarani	|	 Guarani	|	 gn|
|Javanese	|	 Basa Jawa	|	 jv|
|Oriya language	|	 ଓଡ଼ିଆ	|	 or|
|Kinyarwanda	|	 Ikinyarwanda	|	 rw|
|Chinese (Traditional)	|	 繁體中文	|	 zh*|

## Status code table

|`status` | `statusMessage` | Explanation | 
| --- | --- | --- |
|200 | 	OK | Normal |
|227 | 	Parameter Required | Required parameter missing |
|228 |	Do Not Understand | No answer to your question found |
|403 |	Unauthorized | Invalid API Key |
|429 |	Limit Exceeded | Exceeded usage limit |
|500 |	Server error | Server Error |

## Bad word probability
Bad Word Probability is an indicator developed by the SimSimi team to distinguish unhealthy (or malicious) sentences. It is calculated by utilizing various techniques, including advanced deep learning technology that shows excellent performance. For more information, please refer to the following blog post. ([SimSimi Conversation Quality - Bad Word Filter Related Technology](http://blog.simsimi.com/2019/03/blog-post.html))


