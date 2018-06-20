---
ID: 33724
post_title: The `mycroft.conf` file
author: Kathy Reid
post_excerpt: ""
layout: page
permalink: >
  http://mycroft.ai/documentation/mycroft-conf/
published: true
post_date: 2017-12-13 07:51:51
---
# mycroft.conf and web_config_cache.json

- [mycroft.conf and web_config_cache.json](#mycroftconf-and-web_config_cachejson)
  * [What is `mycroft.conf`?](#what-is-mycroft-conf)
  * [What is `web_config_cache.json`?](#what-is-web_config_cachejson)
  * [How are `mycroft.conf` and `web_config_cache.json` updated?](#how-are-mycroft-conf-and-web_config_cache-json-updated)
  * [Where is the `mycroft.conf` file stored?](#where-is-the-mycroft-conf-file-stored)
  * [A look at the inside of `mycroft.conf`](#a-look-at-the-inside-of-mycroft-conf)
  * [Where is the `web_config_cache.json` file stored?](#where-is-the-web_config_cache-json-file-stored)
  * [A look at the inside of `web_config_cache.json`](#a-look-at-the-inside-of-web_config_cache-json)

## What is `mycroft.conf`?

`mycroft.conf` is a [JSON](https://www.json.org/)-formatted file that is saved locally on your Mycroft **Device**, such as Picroft or Mark 1. `mycroft-conf` contains information about the **Device** itself, like what type of **Device** and **Enclosure** it is, as well as information about user preferences. If you haven't specified preferences, then `mycroft.conf` will contain some default values. Your **Device**, and **Skills** installed on your **Device**, use `mycroft.conf` to provide additional functionality. 

## What is `web_config_cache.json`?

`web_config_cache.json` is is a [JSON](https://www.json.org/)-formatted file that is saved locally on your Mycroft **Device**, such as Picroft or Mark 1. `web_config_cache.json` is a cached copy of the settings on  your [home.mycroft.ai](https://home.mycroft.ai) account, such as your _Location_ (which determines _Time Zone_), which _Voice_ you have selected and your preference for _Measurements_ such as temperature and distance. 

Both of these files are regularly used in troubleshooting, so it's useful to know what information they hold, and where they are stored on your **Device**. 

## How are `mycroft.conf` and `web_config_cache.json` updated?

When you update settings at [home.mycroft.ai](https://home.mycroft.ai), your **Device** will periodically pull them down. However, this may not be fast enough to do what you want to do. To refresh your `mycroft.conf` from [home.mycroft.ai](https://home.mycroft.ai), Speak: 

> Hey Mycroft, update config

Mycroft will then pull the files down to your **Device**. 

## Where is the `mycroft.conf` file stored? 

The `mycroft.conf` file is stored in different locations on different **Devices**, including: 

* `/etc/mycroft/mycroft.conf`
* `~/.mycroft/mycroft.conf`

Mycroft implements an order of precedence; settings at a User level override those at a System level. 

## A look at the inside of `mycroft.conf`

Here is an example System level `mycroft.conf` from a Mark 1 **Device**:

```
pi@mark_1:/etc/mycroft $ cat mycroft.conf
{
  "enclosure": {
    "platform": "mycroft_mark_1",
    "platform_build": 9,
    "port": "/dev/ttyAMA0",
    "rate": 9600,
    "timeout": 5,
    "update": true,
    "test": false
  },
  "VolumeSkill": {
    "default_level": 6,
    "min_volume": 0,
    "max_volume": 83
  }
}
```

and here is an example `mycroft.conf` file from Mycroft for Linux, User-level: 

```
{
  // Definition and documentation of all variables used by mycroft-core.
  // 
  // Settings seen here are considered DEFAULT.  Settings can also be
  // overridden at the REMOTE level (set by the user via 
  // https://home.mycroft.ai), at the SYSTEM level (typically in the file
  // '/etc/mycroft/mycroft.conf'), or at the USER level (typically in the
  // file '~/.mycroft/mycroft.conf').
  //
  // The Override: comment indicates at what level (if any) this is 
  // overridden by the system to a value besides the default shown here.

  // Language used for speech-to-text and text-to-speech.
  // Code is a BCP-47 identifier (https://tools.ietf.org/html/bcp47), lowercased
  // TODO: save unmodified, lowercase upon demand
  // Override: none
  "lang": "en-us",
  
  // Measurement units, either 'metric' or 'english'
  // Override: REMOTE
  "system_unit": "metric",
  
  // Time format, either 'half' (e.g. "11:37 pm") or 'full' (e.g. "23:37")
  // Override: REMOTE
  "time_format": "half",
  
  // Date format, either 'MDY' (e.g. "11-29-1978") or 'DMY' (e.g. "29-11-1978")
  // Override: REMOTE
  "date_format": "MDY",

  // Whether to opt in to data collection
  // Override: REMOTE
  "opt_in": false,
  
  // Play a beep when system begins to listen?
  // Override: none
  "confirm_listening": true,
  
  // File locations of sounds to play for system events
  // Override: none
  "sounds": {
    "start_listening": "snd/start_listening.wav",
    "end_listening": "snd/end_listening.wav"
  },
  
  // Mechanism used to play WAV audio files
  // Override: SYSTEM
  "play_wav_cmdline": "paplay %1 --stream-name=mycroft-voice",
  
  // Mechanism used to play MP3 audio files
  // Override: SYSTEM
  "play_mp3_cmdline": "mpg123 %1",
  
  // Location where the system resides
  // NOTE: Although this is set here, an Enclosure can override the value.
  //       For example a mycroft-core running in a car could use the GPS.
  // Override: REMOTE
  "location": {
    "city": {
      "code": "Lawrence",
      "name": "Lawrence",
      "state": {
        "code": "KS",
        "name": "Kansas",
        "country": {
          "code": "US",
          "name": "United States"
        }
      }
    },
    "coordinate": {
      "latitude": 38.971669,
      "longitude": -95.23525
    },
    "timezone": {
      "code": "America/Chicago",
      "name": "Central Standard Time",
      "dstOffset": 3600000,
      "offset": -21600000
    }
  },
  
  // General skill values
  // Override: none
  "skills": {
    // Directory to look for user skills
    "directory": "~/.mycroft/skills",
    // TODO: Old unused kludge, remove from code
    "stop_threshold": 2.0,
    // blacklisted skills to not load
    "blacklisted_skills": ["skill-media", "send_sms", "skill-wolfram-alpha"],
    // priority skills to be loaded first
    "priority_skills": ["skill-pairing"]
  },
  
  // Address of the REMOTE server
  // Override: none
  "server": {
    "url": "https://api.mycroft.ai",
    "version": "v1",
    "update": true,
    "metrics": false
  },
  
  // The mycroft-core messagebus' websocket
  // Override: none
  "websocket": {
    "host": "0.0.0.0",
    "port": 8181,
    "route": "/core",
    "ssl": false
  },
  
  // Settings used by the wake-up-word listener
  // Override: REMOTE
  "listener": {
    "sample_rate": 16000,
    "channels": 1,
    "record_wake_words": false,
    "record_utterances": false,
    "wake_word_upload": {
      "enable": false,
      "server": "mycroft.wickedbroadband.com",
      "port": 1776,
      "user": "precise",
      "folder": "/home/precise/wakewords"
    },
    // In milliseconds
    "phoneme_duration": 120,
    "multiplier": 1.0,
    "energy_ratio": 1.5,
    "wake_word": "hey mycroft",
    "stand_up_word": "wake up"
  },

  // Settings used for any precise wake words
  // Override: none
  "precise": {
    "dist_url": "https://raw.githubusercontent.com/MycroftAI/precise-data/dist/",
    "models_url": "https://raw.githubusercontent.com/MycroftAI/precise-data/models/"
  },

  // Hotword configurations
  "hotwords": {
    "hey mycroft": {
        "module": "pocketsphinx",
        "phonemes": "HH EY . M AY K R AO F T",
        "threshold": 1e-90,
        "lang": "en-us"
        },

    "wake up": {
        "module": "pocketsphinx",
        "phonemes": "W EY K . AH P",
        "threshold": 1e-20,
        "lang": "en-us"
        }
  },

  // Mark 1 enclosure settings
  // Override: SYSTEM (e.g. Picroft)
  "enclosure": {
    // Platform name (e.g. 'Picroft', 'Mark_1'
    // Override: SYSTEM (set by specific enclosures)
    # "platform": "picroft",
    
    // COMM params to the Arduino/faceplate
    "port": "/dev/ttyAMA0",
    "rate": 9600,
    "timeout": 5.0,
    
    // ??
    "update": true,
    
    // Run a self test at bootup?
    "test": false
  },

  // Level of logs to store, one of  "CRITICAL", "ERROR", "WARNING", "INFO", "DEBUG"
  // Override: none
  "log_level": "DEBUG",
  
  // Messagebus types that will NOT be output to logs
  // Override: none
  "ignore_logs": ["enclosure.mouth.viseme", "enclosure.mouth.display"],

  // Settings related to remote sessions
  // Overrride: none
  "session": {
    // Time To Live, in seconds
    "ttl": 180
  },

  // Speech to Text parameters
  // Override: REMOTE
  "stt": {
    // Engine.  Options: "mycroft", "google", "wit", "ibm", "kaldi"
    "module": "mycroft"
    // "kaldi": {
    //   "uri": "http://localhost:8080/client/dynamic/recognize"
    // }
  },

  // Text to Speech parameters
  // Override: REMOTE
  "tts": {
    // Engine.  Options: "mimic", "google", "marytts", "fatts", "espeak", "spdsay"
    "module": "mimic",
    "mimic": {
      "voice": "ap"
    },
    "espeak": {
      "lang": "english-us",
      "voice": "m1"
    }
  },

  "padatious": {
    "intent_cache": "~/.mycroft/intent_cache",
    "train_delay": 4
  },
  // =================================================================
  // All of the follow are specific to particular skills and will soon
  // be removed from this file.
  // =================================================================

  "wifi": {
    "setup": false
  },
  "ConfigurationSkill": {
    "max_delay": 60
  },
  "WikipediaSkill": {
    "max_results": 5,
    "max_phrases": 2
  },
  "WolframAlphaSkill": {
    "api_key": "",
    "proxy": true
  },
  "WeatherSkill": {
    "api_key": "",
    "proxy": true,
    "temperature": "fahrenheit"
  },
  "NPRNewsSkill": {
    "url_rss": "http://www.npr.org/rss/podcast.php?id=500005"
  },
  "AlarmSkill": {
    "filename": "alarm.mp3",
    "max_delay": 600,
    "repeat_time": 20,
    "extended_delay": 60
  },
  "ReminderSkill": {
    "max_delay": 600,
    "repeat_time": 60,
    "extended_delay": 60
  },
  "VolumeSkill": {
    "default_level": 6,
    "min_volume": 0,
    "max_volume": 100
  },
  "AudioRecordSkill": {
    "filename": "/tmp/mycroft-recording.wav",
    "free_disk": 100,
    "max_time": 600,
    "notify_delay": 5,
    "rate": 16000,
    "channels": 1
  },
  "SkillInstallerSkill": {
  },
  "Audio": {
    "backends": {
      "local": {
        "type": "mpg123",
        "active": true
      },
      "vlc": {
        "type": "vlc",
        "active": true
      }
    },
    "default-backend": "local"
  },
  "debug": false
}
```

## Where is the `web_config_cache.json` file stored? 

This file is stored at: 

`/opt/mycroft/web_config_cache.json` 

on the **Device**. 

## A look at the inside of `web_config_cache.json`

Here is an example `web_config_cache.json`. _NOTE: Your settings will be different._

```
{
  "date_format": "DMY",
  "tts": {
    "google": {
      "created_at": 1504481866992,
      "updated_at": 1514794901075
    },
    "module": "mimic",
    "fatts": {
      "created_at": 1504481866991,
      "updated_at": 1514794900939
    },
    "mimic": {
      "created_at": 1504481866989,
      "voice": "ap",
      "updated_at": 1514794900809
    },
    "espeak": {
      "created_at": 1504481866987,
      "updated_at": 1514794900679
    },
    "marytts": {
      "created_at": 1504481866986,
      "updated_at": 1514794900548
    }
  },
  "opt_in": true,
  "created_at": 1504481866955,
  "updated_at": 1514794898083,
  "listener": {
    "energy_ratio": 1.5,
    "created_at": 1504481866996,
    "updated_at": 1514794901398,
    "channels": 1,
    "sample_rate": 16000,
    "multiplier": 1,
    "threshold": 1e-90,
    "phonemes": "HH EY . M AY K R AO F T",
    "wake_word": "hey mycroft"
  },
  "time_format": "full",
  "skills": {
    "directory": "~/.mycroft/skills",
    "created_at": 1504481866994,
    "updated_at": 1514794901226,
    "stop_threshold": 2
  },
  "stt": {
    "google": {
      "credential": {
        "created_at": 1504481866970,
        "updated_at": 1514794899383
      },
      "created_at": 1504481866970,
      "updated_at": 1514794898989
    },
    "ibm": {
      "credential": {
        "created_at": 1504481866958,
        "updated_at": 1514794898334
      },
      "created_at": 1504481866958,
      "updated_at": 1514794898218
    },
    "mycroft": {
      "credential": {
        "created_at": 1504481866965,
        "updated_at": 1514794898856
      },
      "created_at": 1504481866965,
      "updated_at": 1514794898469
    },
    "module": "mycroft",
    "wit": {
      "credential": {
        "created_at": 1504481866981,
        "updated_at": 1514794900417
      },
      "created_at": 1504481866981,
      "updated_at": 1514794900031
    },
    "openstt": {
      "credential": {
        "created_at": 1504481866976,
        "updated_at": 1514794899900
      },
      "created_at": 1504481866976,
      "updated_at": 1514794899521
    }
  },
  "location": {
    "coordinate": {
      "latitude": -38.149918,
      "created_at": 1504500674753,
      "updated_at": 1504500674753,
      "longitude": 144.361719
    },
    "city": {
      "created_at": 1504500674710,
      "state": {
        "country": {
          "created_at": 1486125571309,
          "code": "AU",
          "name": "Australia",
          "updated_at": 1486125571309
        },
        "created_at": 1489950675941,
        "code": "VIC",
        "name": "Victoria",
        "updated_at": 1489950675941
      },
      "code": "Newtown",
      "name": "Newtown",
      "updated_at": 1504500674710
    },
    "created_at": 1504500674709,
    "updated_at": 1504500674709,
    "timezone": {
      "code": "Australia/Melbourne",
      "name": "Australian Eastern Standard Time (Victoria)",
      "dst_offset": 3600000,
      "created_at": 1489950676105,
      "updated_at": 1489950676105,
      "offset": 36000000
    }
  },
  "enclosure": {
    "created_at": 1504481866997,
    "rate": 9600,
    "updated_at": 1514794901541,
    "timeout": 5,
    "port": "/dev/ttyAMA0"
  },
  "system_unit": "metric"
}
```
