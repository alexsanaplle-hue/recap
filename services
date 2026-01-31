
import { GoogleGenAI, Modality } from "@google/genai";

// Custom base64 decoding as per guidelines
function decode(base64: string): Uint8Array {
  const binaryString = atob(base64);
  const len = binaryString.length;
  const bytes = new Uint8Array(len);
  for (let i = 0; i < len; i++) {
    bytes[i] = binaryString.charCodeAt(i);
  }
  return bytes;
}

// Custom PCM audio decoding as per guidelines
async function decodeAudioData(
  data: Uint8Array,
  ctx: AudioContext,
  sampleRate: number,
  numChannels: number,
): Promise<AudioBuffer> {
  const dataInt16 = new Int16Array(data.buffer);
  const frameCount = dataInt16.length / numChannels;
  const buffer = ctx.createBuffer(numChannels, frameCount, sampleRate);

  for (let channel = 0; channel < numChannels; channel++) {
    const channelData = buffer.getChannelData(channel);
    for (let i = 0; i < frameCount; i++) {
      channelData[i] = dataInt16[i * numChannels + channel] / 32768.0;
    }
  }
  return buffer;
}

export async function generateRecapScript(input: string, isFile: boolean = false): Promise<string> {
  // Using gemini-3-pro-preview for high-quality reasoning and localization
  const ai = new GoogleGenAI({ apiKey: process.env.API_KEY });
  
  const response = await ai.models.generateContent({
    model: 'gemini-3-pro-preview',
    contents: `You are an expert Burmese localization specialist and cinematic storyteller.
            Task: Translate and adapt the following ${isFile ? 'source content description (Video or Audio)' : 'YouTube transcript'} into a professional, engaging Burmese (Myanmar) movie recap script.
            
            STRICT REQUIREMENTS:
            1. PRECISE TIME SYNCHRONIZATION: The Burmese script MUST be written such that when spoken, its duration matches the original segment durations provided in the transcript (if any). If no timestamps are provided, estimate a natural pace that matches a standard 1:1 translation length.
            2. NATURAL LOCALIZATION: Do NOT perform a literal word-for-word translation. The script should sound like a native Burmese movie commentary/recap YouTuber. Use idiomatic expressions, engaging transitions, and smooth phrasing that feels natural to a Myanmar audience.
            3. TONE: Exciting, cinematic, and storytelling.
            4. FORMAT: Output only the script in Burmese.
            
            Input: ${input}`,
  });

  return response.text || "Failed to generate script.";
}

export async function generateAudio(text: string, voiceName: string): Promise<string> {
  const ai = new GoogleGenAI({ apiKey: process.env.API_KEY });
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash-preview-tts",
    contents: [{ parts: [{ text: text }] }],
    config: {
      responseModalities: [Modality.AUDIO],
      speechConfig: {
        voiceConfig: {
          prebuiltVoiceConfig: { voiceName: voiceName as any },
        },
      },
    },
  });

  const base64Audio = response.candidates?.[0]?.content?.parts?.[0]?.inlineData?.data;
  if (!base64Audio) throw new Error("No audio data returned from Gemini");

  const audioContext = new (window.AudioContext || (window as any).webkitAudioContext)({ sampleRate: 24000 });
  const decodedData = decode(base64Audio);
  const audioBuffer = await decodeAudioData(decodedData, audioContext, 24000, 1);

  return bufferToWavUrl(audioBuffer);
}

function bufferToWavUrl(buffer: AudioBuffer): string {
  const length = buffer.length * 2;
  const arrayBuffer = new ArrayBuffer(44 + length);
  const view = new DataView(arrayBuffer);

  writeString(view, 0, 'RIFF');
  view.setUint32(4, 36 + length, true);
  writeString(view, 8, 'WAVE');
  writeString(view, 12, 'fmt ');
  view.setUint32(16, 16, true);
  view.setUint16(20, 1, true);
  view.setUint16(22, 1, true);
  view.setUint32(24, buffer.sampleRate, true);
  view.setUint32(28, buffer.sampleRate * 2, true);
  view.setUint16(32, 2, true);
  view.setUint16(34, 16, true);
  writeString(view, 36, 'data');
  view.setUint32(40, length, true);

  const channelData = buffer.getChannelData(0);
  let offset = 44;
  for (let i = 0; i < channelData.length; i++) {
    const sample = Math.max(-1, Math.min(1, channelData[i]));
    view.setInt16(offset, sample < 0 ? sample * 0x8000 : sample * 0x7FFF, true);
    offset += 2;
  }

  const blob = new Blob([arrayBuffer], { type: 'audio/wav' });
  return URL.createObjectURL(blob);
}

function writeString(view: DataView, offset: number, string: string) {
  for (let i = 0; i < string.length; i++) {
    view.setUint8(offset + i, string.charCodeAt(i));
  }
}
