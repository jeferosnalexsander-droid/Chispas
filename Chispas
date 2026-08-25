import "dotenv/config";
import express from "express";
import multer from "multer";
import OpenAI from "openai";
import QRCode from "qrcode";
import fs from "fs";
import path from "path";

const app = express();
const PORT = process.env.PORT || 3000;

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 15 * 1024 * 1024 }
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

app.use(express.json({ limit: "2mb" }));
app.use(express.static("public"));

const instrucciones = `
Tu nombre es Chispas.
Eres una inteligencia artificial moderna, amable, inteligente y seria.
Hablas principalmente español.
Ayudas con programación, estudios, creatividad, preguntas y explicaciones.
Responde de manera clara y útil.
No inventes información.
Mantén las respuestas apropiadas y seguras para adolescentes.
`;

// CHAT
app.post("/api/chat", async (req, res) => {
  try {
    const { message, history = [] } = req.body;

    const respuesta = await openai.responses.create({
      model: "gpt-5",
      input: [
        {
          role: "system",
          content: instrucciones
        },
        ...history.slice(-12),
        {
          role: "user",
          content: message
        }
      ]
    });

    res.json({
      text: respuesta.output_text
    });

  } catch (error) {
    console.error(error);
    res.status(500).json({
      error: "Error conectando con Chispas."
    });
  }
});

// IMÁGENES
app.post("/api/chat-image", upload.single("image"), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({
        error: "No se recibió ninguna imagen."
      });
    }

    const mime = req.file.mimetype;
    const base64 = req.file.buffer.toString("base64");

    const respuesta = await openai.responses.create({
      model: "gpt-5",
      input: [
        {
          role: "user",
          content: [
            {
              type: "input_text",
              text:
                req.body.message ||
                "Analiza esta imagen y explícame qué contiene."
            },
            {
              type: "input_image",
              image_url: `data:${mime};base64,${base64}`
            }
          ]
        }
      ]
    });

    res.json({
      text: respuesta.output_text
    });

  } catch (error) {
    console.error(error);
    res.status(500).json({
      error: "No pude analizar la imagen."
    });
  }
});

// TRANSCRIBIR AUDIO
app.post("/api/transcribe", upload.single("audio"), async (req, res) => {
  try {
    const extension =
      (req.file.originalname || "audio.webm").split(".").pop();

    const archivo = path.join(
      process.cwd(),
      `audio-${Date.now()}.${extension}`
    );

    fs.writeFileSync(archivo, req.file.buffer);

    const transcripcion =
      await openai.audio.transcriptions.create({
        file: fs.createReadStream(archivo),
        model: "gpt-4o-mini-transcribe"
      });

    fs.unlinkSync(archivo);

    res.json({
      text: transcripcion.text
    });

  } catch (error) {
    console.error(error);

    res.status(500).json({
      error: "No pude reconocer el audio."
    });
  }
});

// VOZ DE CHISPAS
app.post("/api/speak", async (req, res) => {
  try {
    const audio = await openai.audio.speech.create({
      model: "gpt-4o-mini-tts",
      voice: "alloy",
      input: req.body.text,
      response_format: "mp3"
    });

    const buffer = Buffer.from(
      await audio.arrayBuffer()
    );

    res.setHeader("Content-Type", "audio/mpeg");
    res.send(buffer);

  } catch (error) {
    console.error(error);

    res.status(500).json({
      error: "No pude generar la voz."
    });
  }
});

// CÓDIGO QR
app.get("/api/qr", async (req, res) => {
  const url =
    process.env.PUBLIC_URL ||
    `http://localhost:${PORT}`;

  const qr = await QRCode.toBuffer(url, {
    width: 400,
    margin: 2
  });

  res.setHeader("Content-Type", "image/png");
  res.send(qr);
});

app.listen(PORT, () => {
  console.log(`
  ✦ CHISPAS AI ✦

  Servidor iniciado:
  http://localhost:${PORT}
  `);
});
