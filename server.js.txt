const express = require("express");
const multer = require("multer");
const cors = require("cors");
const path = require("path");
const fs = require("fs");
const axios = require("axios");

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.static("public"));

// Configure Multer for file uploads
const upload = multer({ dest: "uploads/" });

// Route to handle image upload and AI processing
app.post("/upload", upload.single("image"), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: "No file uploaded" });
  }

  try {
    const inputFilePath = req.file.path;
    const outputFilePath = path.join("public", "output.mp4");

    // Call MadhuAI Custom API
    const apiResponse = await axios.post("https://madhuai-api.com/generate-video", {
      image: fs.readFileSync(inputFilePath, { encoding: "base64" })
    }, {
      headers: { "Authorization": `Bearer YOUR_API_KEY` }
    });

    if (apiResponse.data.videoUrl) {
      fs.writeFileSync(outputFilePath, Buffer.from(apiResponse.data.video, "base64"));
      res.json({ videoUrl: "/output.mp4" });
    } else {
      throw new Error("AI processing failed");
    }
  } catch (error) {
    res.status(500).json({ error: "Failed to process image" });
  }
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
