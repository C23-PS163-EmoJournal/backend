## Emojournal API

Bangkit 2023 Product-based Capstone

Team ID: C23-PS163

(CC) C261DKX4087 – Didi Nur Rahmad –  Universitas Mulawarman

(CC) c153dsx0613 – Raihan Labib Hanif – Universitas Budiluhur

## About

This is the API for Emojournal

## Build with
* [Express](http://expressjs.com/)
* [Node.js](http://nodejs.org/)

## Installation

_Below is an example of how you can instruct your audience on installing and setting up your app._

1. Clone the repo

   ```sh
   git clone https://github.com/C23-PS163-EmoJournal/backend.git
   ```

2. Install NPM packages

   ```sh
   npm install
   ```

3. Run the server
    ```sh
    npm run start
    ```
4. Database
    ```sh
    @google-cloud/storage
    ```

## API endpoints
Base URL : http://localhost:80

Usage
```sh
curl -X POST http://localhost:80/upload
```
Response
```
const upload = async (req, res) => {
  try {
    await processFile(req, res);

    if (!req.file) {
      return res.status(400).send({ message: "Please upload a file!" });
    }

    const blob = bucket.file(req.file.originalname);
    const blobStream = blob.createWriteStream({
      resumable: false,
    });

    blobStream.on("error", (err) => {
      res.status(500).send({ message: err.message });
    });

    blobStream.on("finish", async (data) => {
      const publicUrl = format(
        `https://storage.googleapis.com/${bucket.name}/${blob.name}`
      );

      try {
        await bucket.file(req.file.originalname).makePublic();
      } catch {
        return res.status(500).send({
          message:
            `Uploaded the file successfully: ${req.file.originalname}, but public access is denied!`,
          url: publicUrl,
        });
      }

      res.status(200).send({
        message: "Uploaded the file successfully: " + req.file.originalname,
        url: publicUrl,
      });
    });

    blobStream.end(req.file.buffer);
  } catch (err) {
    console.log(err);

    if (err.code == "LIMIT_FILE_SIZE") {
      return res.status(500).send({
        message: "File size cannot be larger than 2MB!",
      });
    }

    res.status(500).send({
      message: `Could not upload the file: ${req.file.originalname}. ${err}`,
    });
  }
};

```

Usage
```
curl -X GET http://localhost:80/files
```
Response
```
const getListFiles = async (req, res) => {
  try {
    const [files] = await bucket.getFiles();
    let fileInfos = [];

    files.forEach((file) => {
      fileInfos.push({
        name: file.name,
        url: file.metadata.mediaLink,
      });
    });

    res.status(200).send(fileInfos);
  } catch (err) {
    console.log(err);

    res.status(500).send({
      message: "Unable to read list of files!",
    });
  }
};
```

Usage
```
curl -X GET http://localhost:80/files/:name
```
Response
```
const download = async (req, res) => {
  try {
    const [metaData] = await bucket.file(req.params.name).getMetadata();
    res.redirect(metaData.mediaLink);
    
  } catch (err) {
    res.status(500).send({
      message: "Could not download the file. " + err,
    });
  }
};
```

Usage
```sh
curl -X GET http://localhost:80/
```
Response
```
const gethome = (req, res) => {
  res.send("Hello, World!");
};
```
