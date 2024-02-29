# dmarc-visualizer

Analyse and visualize DMARC results using open-source tools.

* [parsedmarc](https://github.com/domainaware/parsedmarc) for parsing DMARC reports,
* [Elasticsearch](https://www.elastic.co/) to store aggregated data.
* [Grafana](https://grafana.com/) to visualize the aggregated reports.

See the full blog post with instructions at https://debricked.com/blog/2020/05/14/analyse-and-visualize-dmarc-results-using-open-source-tools/.

## Screenshot

![Screenshot of Grafana dashboard](/big_screenshot.png?raw=true)


## 動かし方
- dmracのメールにlabel:dmarcをつける

- GASで添付ファイルを取得する

```google-apps-script
/**
 * Searches for emails with a specific label, downloads their attachments, and saves them to Google Drive.
 */
function main() {
  var folderBase = "dmarc"; // 出力先のフォルダ名
  // GMailのメール検索演算子
  // https://support.google.com/mail/answer/7190?hl=ja
  var queryLabel = "label:dmarc"; // GMailの検索タグ

  var folderName = `${folderBase}`;
  saveAttachmentsToDrive(queryLabel, folderName);
}


/**
 * Searches for emails with a specific label, downloads their attachments, and saves them to Google Drive.
 * @param {string} query - The search query to find emails with the desired label and date range.
 * @param {string} folderName - The name of the folder in Google Drive where the attachments will be saved.
 */
function saveAttachmentsToDrive(query, folderName) {
  var folder = createFolderByPath(folderName);
  deleteAllFilesInFolder(folder);

  // Save attachments from emails matching the search query
  var threads = GmailApp.search(query);
  for (var i = 0; i < threads.length; i++) {
    var messages = threads[i].getMessages();
    for (var j = 0; j < messages.length; j++) {
      Logger.log(messages[j].getSubject() + " (" + messages[j].getDate() + ")");
      var attachments = messages[j].getAttachments();
      for (var k = 0; k < attachments.length; k++) {
        var attachment = attachments[k];
        folder.createFile(attachment);
      }
    }
  }
}

/**
 * Creates a folder in Google Drive recursively.
 * @param {string} path - The path of the folder to create, separated by forward slashes.
 * @returns {Folder} - The created folder object.
 */
function createFolderByPath(path) {
  var folders = path.split('/');
  var folder = DriveApp.getRootFolder();
  for (var i = 0; i < folders.length; i++) {
    var subFolders = folder.getFoldersByName(folders[i]);
    if (subFolders.hasNext()) {
      folder = subFolders.next();
    } else {
      folder = folder.createFolder(folders[i]);
    }
  }
  return folder;
}


/**
 * Deletes all files in a given folder.
 * @param {Folder} folder - The folder in Google Drive where the files will be deleted.
 */
function deleteAllFilesInFolder(folder) {
  if (folder) {
    var files = folder.getFiles();
    while (files.hasNext()) {
      var file = files.next();
      file.setTrashed(true);
    }
  }
}
```
- filesにあるファイルを配置

- 立ち上げる
```bassh
docker compose up
```

- tips
parsedmarcはジョブ実行なので終わったら消える、実行中は`docker compose ps`で確認する
