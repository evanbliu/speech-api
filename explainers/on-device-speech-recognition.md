# Explainer: On-Device Speech Recognition for the Web Speech API

## Introduction

The Web Speech API is a powerful browser feature that enables applications to perform speech recognition. Traditionally, this functionality relies on sending audio data to cloud-based services for recognition. While this approach is effective, it has certain drawbacks:

- **Privacy concerns:** Raw and transcribed audio is transmitted over the network.
- **Latency issues:** Users may experience delays due to network communication.
- **Offline limitations:** Speech recognition does not work without an internet connection.

To address these issues, we introduce **on-device speech recognition capabilities** as part of the Web Speech API. This enhancement allows speech recognition to run locally on user devices, providing a faster, more private, and offline-compatible experience.

## Why Use On-Device Speech Recognition?

### 1. **Privacy**
On-device processing ensures that neither raw audio nor transcriptions leave the user's device, enhancing data security and user trust.

### 2. **Performance**
Local processing reduces latency, providing a smoother and faster user experience.

### 3. **Offline Functionality**
Applications can offer speech recognition capabilities even without an active internet connection, increasing their utility in remote or low-connectivity environments.

## Example use cases
### 1. Company with data residency requirements
Websites with strict data residency requirements (i.e., regulatory, legal, or company policy) can ensure that audio data remains on the user's device and is not sent over the network for processing. This is particularly crucial for compliance with regulations like GDPR, which considers voice as personally identifiable information (PII) as voice recordings can reveal information about an individual's gender, ethnic origin, or even potential health conditions. On-device processing significantly enhances user privacy by minimizing the exposure of sensitive voice data.

### 2. Video conferencing service with strict performance requirements (e.g. meet.google.com)
Some websites would only adopt the Web Speech API if it meets strict performance requirements. On-device speech recognition may provide better accuracy and latency as well as provide additional features (e.g. contextual biasing) that may not be available by the cloud-based service used by the user agent. In the event on-device speech recognition is not available, these websites may elect to use an alternative cloud-based speech recognition provider that meet these requirements instead of the default one provided by the user agent.

### 3. Educational website (e.g. khanacademy.org)
Applications that need to function in unreliable or offline network conditions—such as voice-based productivity tools, educational software, or accessibility features—benefit from on-device speech recognition. This enables uninterrupted functionality during flights, remote travel, or in areas with limited connectivity. When on-device recognition is unavailable, a website can choose to hide the UI or gracefully degrade functionality to maintain a coherent user experience.

## New API Components

This enhancement introduces one new attribute to the `SpeechRecognition` interface and two new static methods for managing on-device capabilities.

### 1. `processLocally` Attribute
The `processLocally` boolean attribute on a `SpeechRecognition` instance allows developers to require that speech recognition be performed locally on the user's device.

- When set to `true`, the recognition session **must** be processed locally. If on-device recognition is not available for the specified language, the session will fail with a `service-not-allowed` error.
- When `false` (the default), the user agent is free to use either local or cloud-based recognition.

#### Example Usage
```javascript
const recognition = new SpeechRecognition();
recognition.lang = 'en-US';
recognition.processLocally = true; // Require on-device speech recognition.

recognition.onerror = (event) => {
  if (event.error === 'service-not-allowed') {
    console.error('On-device recognition is not available for the selected language, or the request was denied.');
  }
};

recognition.start();
```

### 2. `Promise<boolean> available(SpeechRecognitionOptions options)`
The static `SpeechRecognition.available(options)` method allows developers to check the availability of speech recognition for a given set of languages and processing preferences. It returns a `Promise` that resolves with an `AvailabilityStatus` string.

#### Example Usage
```javascript
const options = {
  langs: ['en-US'],
  processLocally: true // Check for on-device availability
};

SpeechRecognition.available(options).then((status) => {
  console.log(`On-device availability for ${options.langs.join(', ')}: ${status}`);
  if (status === 'available') {
    console.log('Ready to use on-device recognition.');
  } else if (status === 'downloadable') {
    console.log('On-device recognition can be installed.');
  }
});
```

### 2. `Promise<boolean> install(SpeechRecognitionOptions options)`
This method install the resources required for on-device speech recognition for the given BCP-47 language codes. The installation process may download and configure necessary language models.

#### Example Usage
```javascript
const options = {
  langs: ['en-US'],
  processLocally: true
};
SpeechRecognition.install(options).then((success) => {
    if (success) {
        console.log('On-device speech recognition resources installed successfully.');
    } else {
        console.error('Unable to install on-device speech recognition.');
    }
});
```

## Supported languages
The availability of on-device speech recognition languages is user-agent dependent. As an example, Google Chrome supports the following languages for on-device recognition:
* de-DE (German, Germany)
* en-US (English, United States)
* es-ES (Spanish, Spain)
* fr-FR (French, France)
* hi-IN (Hindi, India)
* id-ID (Indonesian, Indonesia)
* it-IT (Italian, Italy)
* ja-JP (Japanese, Japan)
* ko-KR (Korean, South Korea)
* pl-PL (Polish, Poland)
* pt-BR (Portuguese, Brazil)
* ru-RU (Russian, Russia)
* th-TH (Thai, Thailand)
* tr-TR (Turkish, Turkey)
* vi-VN (Vietnamese, Vietnam)
* zh-CN (Chinese, Mandarin, Simplified)
* zh-TW (Chinese, Mandarin, Traditional)

## Privacy considerations
To reduce the risk of fingerprinting, user agents must implementing privacy-preserving countermeasures. The Web Speech API will employ the same masking techniques used by the [Web Translation API](https://github.com/webmachinelearning/writing-assistance-apis/pull/47).

## Conclusion
The addition of on-device speech recognition capabilities to the Web Speech API marks a significant step forward in creating more private, performant, and accessible web applications. By leveraging these new methods, developers can enhance user experiences while addressing key concerns around privacy and connectivity.