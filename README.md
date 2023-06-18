# BookmarkletToGetTranscriptOfYouTube

- Just open a Youtube video in your browser and run the bookmarklet to copy the subtitle information to the clipboard.
  - Youtubeの動画をブラウザで開いて、ブックマークレットを実行するだけでクリップボードに字幕情報がコピーされます。
- It's running in the GoogleChrome environment on a Mac (as of 2023-06-19).
  - MacのGoogleChrome環境で動いてます（2023-06-19現在）

```javascript
javascript:(function() {
    var scripts = Array.from(document.getElementsByTagName("script")).find(function(element) {
        return element.textContent.includes("timedtext");
    });
    if (scripts) {
        var regex = /%7B"baseUrl":"(https:%5C/%5C/www%5C.youtube%5C.com%5C/api%5C/timedtext%5B%5E"%5D*)"/;
        var matches = scripts.textContent.match(regex);
        if (matches) {
            var url = matches[1].replace(/%5C%5Cu0026/g, "&");
            fetch(url)
                .then(function(response) {
                    return response.text();
                })
                .then(function(data) {
                    var xmlDoc = new DOMParser().parseFromString(data, "text/xml");
                    var textElements = xmlDoc.getElementsByTagName("text");
                    var transcript = "";
                    for (var i = 0; i < textElements.length; i++) {
                        transcript += decodeEntities(textElements[i].textContent) + " ";
                    }
                    var textarea = document.createElement("textarea");
                    textarea.value = transcript;
                    document.body.appendChild(textarea);
                    textarea.select();
                    document.execCommand("copy");
                    document.body.removeChild(textarea);
                    alert("Transcript has been copied to the clipboard.");
                })
                .catch(function(error) {
                    console.log(error);
                    alert("Failed to fetch the transcript: " + error);
                });
        } else {
            alert("Could not find timed text URL.");
        }
    } else {
        alert("Could not find timed text data.");
    }
})();

function decodeEntities(text) {
    var textarea = document.createElement('textarea');
    textarea.innerHTML = text;
    return textarea.value;
}
```
