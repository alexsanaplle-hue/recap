
import React, { useState } from 'react';
import Header from './components/Header';
import ProcessingOverlay from './components/ProcessingOverlay';
import AudioPlayer from './components/AudioPlayer';
import { ProcessingStatus, RecapResult, AVAILABLE_VOICES, VoiceOption } from './types';
import { generateRecapScript, generateAudio } from './services/geminiService';
import { FileText, Video, AlertCircle, RefreshCw, Send, CheckCircle2, ChevronRight, User, Users, Mic, Speaker } from 'lucide-react';

const App: React.FC = () => {
  const [inputMode, setInputMode] = useState<'text' | 'video' | 'audio' | 'tts'>('text');
  const [transcript, setTranscript] = useState('');
  const [selectedVoice, setSelectedVoice] = useState<VoiceOption>(AVAILABLE_VOICES[0]);
  const [status, setStatus] = useState<ProcessingStatus>(ProcessingStatus.IDLE);
  const [result, setResult] = useState<RecapResult | null>(null);
  const [error, setError] = useState<string | null>(null);

  const handleProcess = async () => {
    if (!transcript && (inputMode === 'text' || inputMode === 'tts')) {
      setError(`Please ${inputMode === 'tts' ? 'enter text' : 'paste a transcript'} first!`);
      return;
    }

    setStatus(ProcessingStatus.PROCESSING);
    setError(null);

    try {
      let finalScript = '';
      
      if (inputMode === 'tts') {
        // Skip script generation for direct TTS
        finalScript = transcript;
      } else {
        setStatus(ProcessingStatus.GENERATING_SCRIPT);
        // Pass a boolean indicating if it's a file-based input (video or audio)
        const isFileMode = inputMode !== 'text';
        finalScript = await generateRecapScript(transcript, isFileMode);
      }
      
      setStatus(ProcessingStatus.GENERATING_AUDIO);
      const audioUrl = await generateAudio(finalScript, selectedVoice.id);

      setResult({
        script: finalScript,
        audioUrl,
        originalFileName: inputMode === 'text' ? 'Pasted Transcript' : 
                         (inputMode === 'video' ? 'Uploaded Video' : 
                         (inputMode === 'audio' ? 'Uploaded Audio' : 'Direct Text Input')),
        voiceSelected: `${selectedVoice.label} (${selectedVoice.gender})`
      });
      setStatus(ProcessingStatus.COMPLETED);
    } catch (err: any) {
      console.error(err);
      setError(err.message || "An unexpected error occurred. Please try again.");
      setStatus(ProcessingStatus.ERROR);
    }
  };

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      if (file.size > 500 * 1024 * 1024) {
        setError("File is too large! Max 500MB.");
        return;
      }
      const prefix = file.type.startsWith('video/') ? 'Video File' : 'Audio File';
      setTranscript(`${prefix}: ${file.name}. Size: ${(file.size / 1024 / 1024).toFixed(2)}MB. Type: ${file.type}`);
      setError(null);
    }
  };

  const reset = () => {
    setStatus(ProcessingStatus.IDLE);
    setResult(null);
    setError(null);
    setTranscript('');
  };

  return (
    <div className="min-h-screen pb-20 px-4 md:px-8 max-w-6xl mx-auto">
      <Header />
      <ProcessingOverlay status={status} />

      <main className="space-y-8 animate-in fade-in slide-in-from-bottom-4 duration-1000">
        {status === ProcessingStatus.IDLE || status === ProcessingStatus.ERROR ? (
          <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
            {/* Input Side */}
            <div className="lg:col-span-2 bg-slate-900 border border-slate-800 rounded-3xl p-6 md:p-10 shadow-xl space-y-6">
              <div className="flex flex-col space-y-4">
                <div className="flex p-1 bg-slate-950 rounded-2xl w-full flex-wrap gap-1">
                  <button
                    onClick={() => { setInputMode('text'); setTranscript(''); }}
                    className={`flex-1 sm:flex-none px-4 py-2.5 rounded-xl flex items-center justify-center space-x-2 transition-all ${
                      inputMode === 'text' ? 'bg-slate-800 text-white shadow-lg' : 'text-slate-500 hover:text-slate-300'
                    }`}
                  >
                    <FileText className="w-4 h-4" />
                    <span>Recap Script</span>
                  </button>
                  <button
                    onClick={() => { setInputMode('video'); setTranscript(''); }}
                    className={`flex-1 sm:flex-none px-4 py-2.5 rounded-xl flex items-center justify-center space-x-2 transition-all ${
                      inputMode === 'video' ? 'bg-slate-800 text-white shadow-lg' : 'text-slate-500 hover:text-slate-300'
                    }`}
                  >
                    <Video className="w-4 h-4" />
                    <span>Video</span>
                  </button>
                  <button
                    onClick={() => { setInputMode('audio'); setTranscript(''); }}
                    className={`flex-1 sm:flex-none px-4 py-2.5 rounded-xl flex items-center justify-center space-x-2 transition-all ${
                      inputMode === 'audio' ? 'bg-slate-800 text-white shadow-lg' : 'text-slate-500 hover:text-slate-300'
                    }`}
                  >
                    <Mic className="w-4 h-4" />
                    <span>Audio</span>
                  </button>
                  <button
                    onClick={() => { setInputMode('tts'); setTranscript(''); }}
                    className={`flex-1 sm:flex-none px-4 py-2.5 rounded-xl flex items-center justify-center space-x-2 transition-all ${
                      inputMode === 'tts' ? 'bg-slate-800 text-white shadow-lg' : 'text-slate-500 hover:text-slate-300'
                    }`}
                  >
                    <Speaker className="w-4 h-4" />
                    <span>Direct TTS</span>
                  </button>
                </div>

                {inputMode === 'text' || inputMode === 'tts' ? (
                  <div className="space-y-2">
                    <label className="text-sm font-medium text-slate-400 ml-1">
                      {inputMode === 'text' 
                        ? 'Paste Transcript (Include timestamps like [00:00-00:05] for precise sync)' 
                        : 'Enter Burmese Text (Directly converted to speech)'}
                    </label>
                    <textarea
                      value={transcript}
                      onChange={(e) => setTranscript(e.target.value)}
                      placeholder={inputMode === 'text' 
                        ? "Example: [00:00-00:05] The hero enters the dark room..." 
                        : "မင်္ဂလာပါ... ဒီနေ့မှာတော့ ကျွန်တော်တို့ရဲ့ ဇာတ်လမ်းကို စတင်လိုက်ရအောင်..."}
                      className="w-full h-80 bg-slate-950 border border-slate-800 rounded-2xl p-6 focus:ring-2 focus:ring-red-500 focus:border-transparent transition-all outline-none resize-none text-slate-200 font-mono text-sm"
                    />
                  </div>
                ) : (
                  <div className="space-y-4">
                    <label className="text-sm font-medium text-slate-400 ml-1">
                      Upload {inputMode === 'video' ? 'Movie Clip' : 'Audio Track'} (Max 500MB)
                    </label>
                    <div className="border-2 border-dashed border-slate-800 hover:border-red-500/50 transition-colors rounded-2xl p-12 flex flex-col items-center justify-center text-center space-y-4 cursor-pointer relative">
                      <input 
                        type="file" 
                        accept={inputMode === 'video' ? "video/*" : "audio/*"} 
                        onChange={handleFileChange} 
                        className="absolute inset-0 opacity-0 cursor-pointer" 
                      />
                      <div className="p-4 bg-slate-950 rounded-full">
                        {inputMode === 'video' ? <Video className="w-8 h-8 text-slate-500" /> : <Mic className="w-8 h-8 text-slate-500" />}
                      </div>
                      <div>
                        <p className="text-lg font-medium text-white">
                          {transcript.includes('File: ') ? transcript.split('Size:')[0] : 'Click to upload or drag and drop'}
                        </p>
                        <p className="text-sm text-slate-500">
                          {inputMode === 'video' ? 'MP4, MOV, or WEBM' : 'MP3, WAV, or AAC'} up to 500MB
                        </p>
                      </div>
                    </div>
                  </div>
                )}
              </div>

              {error && (
                <div className="flex items-center space-x-2 p-4 bg-red-500/10 border border-red-500/20 rounded-xl text-red-400 text-sm">
                  <AlertCircle className="w-4 h-4 shrink-0" />
                  <span>{error}</span>
                </div>
              )}
            </div>

            {/* Voice Selection Column */}
            <div className="bg-slate-900 border border-slate-800 rounded-3xl p-6 md:p-8 shadow-xl space-y-6">
              <h3 className="text-lg font-bold text-white flex items-center space-x-2">
                <Users className="w-5 h-5 text-red-500" />
                <span>Select AI Voice</span>
              </h3>
              <div className="space-y-3 max-h-[400px] overflow-y-auto pr-2 custom-scrollbar">
                {AVAILABLE_VOICES.map((voice) => (
                  <div
                    key={voice.id}
                    onClick={() => setSelectedVoice(voice)}
                    className={`p-4 rounded-2xl border cursor-pointer transition-all ${
                      selectedVoice.id === voice.id
                        ? 'bg-red-500/10 border-red-500'
                        : 'bg-slate-950 border-slate-800 hover:border-slate-700'
                    }`}
                  >
                    <div className="flex items-center justify-between">
                      <div className="flex items-center space-x-3">
                        <div className={`p-2 rounded-lg ${selectedVoice.id === voice.id ? 'bg-red-500 text-white' : 'bg-slate-800 text-slate-400'}`}>
                          <User className="w-4 h-4" />
                        </div>
                        <div>
                          <p className="font-bold text-sm text-white">{voice.label}</p>
                          <p className="text-xs text-slate-500">{voice.gender} • {voice.description}</p>
                        </div>
                      </div>
                      {selectedVoice.id === voice.id && (
                        <div className="w-2 h-2 rounded-full bg-red-500 shadow-[0_0_8px_rgba(239,68,68,0.8)]" />
                      )}
                    </div>
                  </div>
                ))}
              </div>

              <button
                onClick={handleProcess}
                className="w-full py-4 bg-gradient-to-r from-red-600 to-amber-600 hover:from-red-500 hover:to-amber-500 text-white font-bold rounded-2xl shadow-lg shadow-red-900/20 flex items-center justify-center space-x-2 transition-all active:scale-[0.98]"
              >
                <Send className="w-5 h-5" />
                <span>{inputMode === 'tts' ? 'Convert to Speech' : 'Generate Synced Recap'}</span>
              </button>
            </div>
          </div>
        ) : result && status === ProcessingStatus.COMPLETED ? (
          <div className="space-y-8 animate-in zoom-in-95 duration-500">
            <div className="flex items-center justify-between p-6 bg-green-500/10 border border-green-500/20 rounded-3xl">
              <div className="flex items-center space-x-4">
                <div className="p-2 bg-green-500 rounded-full text-slate-950">
                  <CheckCircle2 className="w-6 h-6" />
                </div>
                <div>
                  <h3 className="font-bold text-white text-lg">
                    {inputMode === 'tts' ? 'Speech Conversion Complete!' : 'Synced Recap Ready!'}
                  </h3>
                  <p className="text-green-500/80 text-sm">Processed using Gemini's high-fidelity TTS engine.</p>
                </div>
              </div>
              <button onClick={reset} className="flex items-center space-x-2 text-sm text-slate-400 hover:text-white transition-colors">
                <RefreshCw className="w-4 h-4" />
                <span>Start Again</span>
              </button>
            </div>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
              <div className="lg:col-span-2 space-y-4">
                <div className="bg-slate-900 border border-slate-800 rounded-3xl overflow-hidden shadow-2xl">
                  <div className="p-6 border-b border-slate-800 bg-slate-800/20 flex items-center justify-between">
                    <h3 className="font-bold text-white flex items-center space-x-2">
                      <FileText className="w-5 h-5 text-amber-500" />
                      <span>{inputMode === 'tts' ? 'Source Text' : 'Burmese Master Script'}</span>
                    </h3>
                  </div>
                  <div className="p-8 h-[550px] overflow-y-auto text-slate-200 leading-[2.2] font-['Padauk'] text-xl whitespace-pre-wrap selection:bg-red-500/30">
                    {result.script}
                  </div>
                </div>
              </div>

              <div className="space-y-6">
                <div className="bg-slate-900 border border-slate-800 rounded-3xl p-6 shadow-xl space-y-6">
                  <h3 className="font-bold text-white">Audio Output</h3>
                  {result.audioUrl && <AudioPlayer url={result.audioUrl} />}
                  
                  <div className="pt-4 border-t border-slate-800 space-y-4">
                    <h4 className="text-xs font-bold text-slate-500 uppercase tracking-widest">Metadata</h4>
                    <div className="space-y-3">
                      <div className="flex items-center justify-between text-sm">
                        <span className="text-slate-500">AI Voice</span>
                        <span className="text-red-400 font-medium">{result.voiceSelected}</span>
                      </div>
                      <div className="flex items-center justify-between text-sm">
                        <span className="text-slate-500">Mode</span>
                        <span className="text-slate-300">{inputMode === 'tts' ? 'Direct TTS' : 'Synced Recap'}</span>
                      </div>
                      <div className="flex items-center justify-between text-sm">
                        <span className="text-slate-500">Engine</span>
                        <span className="text-slate-300">Gemini 2.5 TTS</span>
                      </div>
                    </div>
                  </div>

                  <button
                    className="w-full py-4 bg-slate-800 hover:bg-slate-700 text-white font-bold rounded-2xl flex items-center justify-center space-x-2 transition-all"
                    onClick={() => {
                      navigator.clipboard.writeText(result.script);
                      alert("Content copied to clipboard!");
                    }}
                  >
                    <span>Copy Text</span>
                  </button>
                </div>

                <div className="bg-gradient-to-br from-red-600 to-amber-700 rounded-3xl p-6 text-white space-y-4 shadow-xl">
                  <h3 className="font-bold">Usage Tip</h3>
                  <p className="text-sm text-red-50 text-balance">
                    {inputMode === 'tts' 
                      ? 'You can use this direct speech output for voiceovers in any project. The audio is provided in standard WAV format.' 
                      : 'This audio is generated in 24kHz HQ. Overlay this track on your movie clip for an instant professional recap!'}
                  </p>
                </div>
              </div>
            </div>
          </div>
        ) : null}
      </main>
    </div>
  );
};

export default App;
