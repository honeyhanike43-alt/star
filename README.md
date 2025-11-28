import React, { useState, useCallback, useRef, useEffect } from 'react';
import StarBackground from './components/StarBackground';
import LotteryContainer from './components/LotteryContainer';
import ResultCard from './components/ResultCard';
import { CONSTELLATIONS } from './constants';
import { Constellation } from './types';

// Audio Assets
const AUDIO_SOURCES = {
  // Deep, ambient, mysterious
  bgm: "https://assets.mixkit.co/music/preview/mixkit-deep-meditation-108.mp3",
  // Ethereal, magical sparkle
  draw: "https://assets.mixkit.co/sfx/preview/mixkit-magic-wind-sparkle-2656.mp3",
  // Deep revelation/impact
  reveal: "https://assets.mixkit.co/sfx/preview/mixkit-cinematic-deep-impact-hit-2457.mp3"
};

type AppState = 'intro' | 'drawing' | 'result';

const App: React.FC = () => {
  const [appState, setAppState] = useState<AppState>('intro');
  const [result, setResult] = useState<Constellation | null>(null);
  
  // Audio State & Refs
  const [isMuted, setIsMuted] = useState(false);
  const [hasInteracted, setHasInteracted] = useState(false);
  const bgmRef = useRef<HTMLAudioElement>(null);
  const drawSfxRef = useRef<HTMLAudioElement>(null);
  const revealSfxRef = useRef<HTMLAudioElement>(null);

  // Initialize Audio Settings
  useEffect(() => {
    if (bgmRef.current) {
      bgmRef.current.volume = 0.3; // Lower volume for BGM so it's not overwhelming
    }
    if (drawSfxRef.current) drawSfxRef.current.volume = 0.6;
    if (revealSfxRef.current) revealSfxRef.current.volume = 0.8;
  }, []);

  // Handle BGM Playback
  useEffect(() => {
    const playBgm = async () => {
      if (bgmRef.current && !isMuted) {
        try {
          await bgmRef.current.play();
        } catch (err) {
          // Autoplay was prevented by the browser
          console.log("Autoplay prevented. Waiting for interaction.");
        }
      } else if (bgmRef.current && isMuted) {
        bgmRef.current.pause();
      }
    };
    playBgm();
  }, [isMuted, hasInteracted]);

  const toggleMute = () => {
    setIsMuted(!isMuted);
    setHasInteracted(true);
  };

  const playSfx = (ref: React.RefObject<HTMLAudioElement | null>) => {
    if (!isMuted && ref.current) {
      ref.current.currentTime = 0;
      ref.current.play().catch(() => {});
    }
  };

  // Random drawing logic
  const handleDraw = useCallback(() => {
    if (appState === 'drawing') return; // Prevent double clicks
    
    // User interaction confirmed
    setHasInteracted(true);
    
    setAppState('drawing');
    playSfx(drawSfxRef);

    // Simulate the "ritual" duration
    setTimeout(() => {
      const randomIndex = Math.floor(Math.random() * CONSTELLATIONS.length);
      const selectedConstellation = CONSTELLATIONS[randomIndex];
      setResult(selectedConstellation);
      setAppState('result');
      playSfx(revealSfxRef);
    }, 2000); // 2 seconds of animation
  }, [appState, isMuted]);

  const handleReset = useCallback(() => {
    setResult(null);
    setAppState('intro');
  }, []);

  return (
    <div className="relative min-h-screen w-full flex flex-col bg-celestial-dark font-serif overflow-x-hidden selection:bg-mystic-gold selection:text-celestial-dark">
      
      {/* Audio Elements */}
      <audio ref={bgmRef} src={AUDIO_SOURCES.bgm} loop preload="auto" />
      <audio ref={drawSfxRef} src={AUDIO_SOURCES.draw} preload="auto" />
      <audio ref={revealSfxRef} src={AUDIO_SOURCES.reveal} preload="auto" />

      {/* Background Layer */}
      <StarBackground />

      {/* Header / Nav (Static) */}
      <header className="absolute top-0 w-full p-6 z-20 flex justify-between items-center opacity-80 hover:opacity-100 transition-opacity">
        <div className="flex flex-col">
            <div className="text-mystic-gold font-display text-sm md:text-base tracking-[0.2em] uppercase border border-mystic-gold/30 px-4 py-2 inline-block">
                The Celestial 28
            </div>
            <div className="hidden md:block text-slate-500 text-xs tracking-widest mt-1 ml-1">
                天界二十八星宿
            </div>
        </div>

        {/* Audio Toggle Button */}
        <button 
          onClick={toggleMute}
          className="group flex items-center justify-center w-10 h-10 border border-mystic-gold/50 rounded-full hover:bg-mystic-gold/10 hover:border-mystic-gold transition-all duration-300"
          title={isMuted ? "Unmute Sound" : "Mute Sound"}
        >
          {isMuted ? (
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-5 h-5 text-slate-400 group-hover:text-mystic-gold">
              <path strokeLinecap="round" strokeLinejoin="round" d="M17.25 9.75 19.5 12m0 0 2.25 2.25M19.5 12l2.25-2.25M19.5 12l-2.25 2.25m-10.5-6 4.72-4.72a.75.75 0 0 1 1.28.53v15.88a.75.75 0 0 1-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.009 9.009 0 0 1 2.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75Z" />
            </svg>
          ) : (
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-5 h-5 text-mystic-gold animate-pulse">
              <path strokeLinecap="round" strokeLinejoin="round" d="M19.114 5.636a9 9 0 0 1 0 12.728M16.463 8.288a5.25 5.25 0 0 1 0 7.424M6.75 8.25l4.72-4.72a.75.75 0 0 1 1.28.53v15.88a.75.75 0 0 1-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.009 9.009 0 0 1 2.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75Z" />
            </svg>
          )}
        </button>
      </header>

      {/* Main Content Area */}
      <main className="flex-grow flex items-center justify-center p-4 md:p-8 z-10">
        
        {appState === 'intro' && (
          <div className="animate-fade-in w-full">
            <LotteryContainer onDraw={handleDraw} isShaking={false} />
          </div>
        )}

        {appState === 'drawing' && (
          <div className="w-full">
            <LotteryContainer onDraw={() => {}} isShaking={true} />
          </div>
        )}

        {appState === 'result' && result && (
          <ResultCard data={result} onReset={handleReset} />
        )}

      </main>

      {/* Footer */}
      <footer className="absolute bottom-0 w-full p-6 text-center z-20">
        <p className="text-slate-600 text-xs uppercase tracking-widest">
          &copy; {new Date().getFullYear()} Ancient Star Divination
        </p>
      </footer>

    </div>
  );
};

export default App;
import React from 'react';

interface LotteryContainerProps {
  onDraw: () => void;
  isShaking: boolean;
}

const LotteryContainer: React.FC<LotteryContainerProps> = ({ onDraw, isShaking }) => {
  return (
    <div className="flex flex-col items-center justify-center min-h-[50vh] z-10 relative">
      <div className="relative group cursor-pointer" onClick={onDraw}>
        {/* Glow Effect behind the container */}
        <div className="absolute inset-0 bg-mystic-gold rounded-full blur-[60px] opacity-20 group-hover:opacity-40 transition-opacity duration-700"></div>

        {/* The Container SVG */}
        <div className={`relative w-64 h-80 transition-transform duration-300 ${isShaking ? 'animate-shake' : 'animate-float group-hover:scale-105'}`}>
          <svg viewBox="0 0 200 300" fill="none" xmlns="http://www.w3.org/2000/svg" className="w-full h-full drop-shadow-2xl">
            {/* Cylinder Body */}
            <path d="M40 50 C40 30, 160 30, 160 50 L150 250 C150 280, 50 280, 50 250 Z" 
              className="fill-slate-800 stroke-mystic-gold stroke-2" />
            
            {/* Lid/Rim Inner */}
            <ellipse cx="100" cy="50" rx="60" ry="15" className="fill-slate-900 stroke-mystic-gold stroke-2 opacity-80" />
            
            {/* Decorative Patterns */}
            <path d="M60 100 Q100 130 140 100" className="stroke-mystic-gold stroke-1 fill-none opacity-50" />
            <path d="M55 150 Q100 180 145 150" className="stroke-mystic-gold stroke-1 fill-none opacity-50" />
            <path d="M52 200 Q100 230 148 200" className="stroke-mystic-gold stroke-1 fill-none opacity-50" />

            {/* Sticks protruding */}
            <g className="transition-transform duration-500 origin-bottom" style={{ transform: isShaking ? 'translateY(-10px)' : 'translateY(0)' }}>
              <rect x="90" y="20" width="6" height="60" className="fill-amber-700 stroke-amber-900" transform="rotate(-10 93 80)" />
              <rect x="104" y="15" width="6" height="70" className="fill-amber-600 stroke-amber-900" transform="rotate(5 107 85)" />
              <rect x="95" y="25" width="6" height="55" className="fill-amber-800 stroke-amber-900" transform="rotate(15 98 80)" />
            </g>

            {/* Chinese Character for 'Divination' */}
            <text x="100" y="160" textAnchor="middle" className="fill-mystic-gold font-serif text-4xl opacity-90 select-none" style={{ filter: 'drop-shadow(0px 0px 5px rgba(212, 175, 55, 0.5))' }}>
              簽
            </text>
          </svg>
        </div>
      </div>

      <div className="mt-12 text-center space-y-4">
        <h2 className="text-2xl md:text-3xl font-display text-mystic-gold tracking-widest uppercase">
          {isShaking ? "The Stars Align..." : "Consult the Stars"}
        </h2>
        <p className="text-slate-400 font-serif italic max-w-md">
          {isShaking 
            ? "Focus on your question as the heavens shift." 
            : "Click the sacred vessel to reveal your celestial guardian."}
        </p>
      </div>
      
      {/* Interaction Button (Alternative to clicking the pot) */}
      {!isShaking && (
        <button 
          onClick={onDraw}
          className="mt-8 px-8 py-3 bg-transparent border border-mystic-gold text-mystic-gold hover:bg-mystic-gold hover:text-slate-900 transition-all duration-300 font-display tracking-widest rounded-sm uppercase text-sm"
        >
          Draw Fate
        </button>
      )}
    </div>
  );
};

export default LotteryContainer;
import React, { useEffect, useState, useRef } from 'react';
import { Constellation } from '../types';
import { GoogleGenAI } from "@google/genai";

interface ResultCardProps {
  data: Constellation;
  onReset: () => void;
}

const ResultCard: React.FC<ResultCardProps> = ({ data, onReset }) => {
  const [imageUrl, setImageUrl] = useState<string | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const generatedRef = useRef<boolean>(false);

  useEffect(() => {
    // Prevent double generation in StrictMode
    if (generatedRef.current) return;
    generatedRef.current = true;

    const generateImage = async () => {
      setLoading(true);
      try {
        if (!process.env.API_KEY) {
            console.warn("No API Key found. Using fallback image.");
            setImageUrl(`https://picsum.photos/seed/${data.id * 123}/800/1000`);
            setLoading(false);
            return;
        }

        const ai = new GoogleGenAI({ apiKey: process.env.API_KEY });
        
        const stylePrompt = `
          Vintage hand-drawn illustration style divination card. 
          Western magic, alchemy, and Gothic fantasy aesthetic. 
          Low saturation classical oil painting palette: deep red, rust gold, dark green, bronze. 
          Visible brushstrokes, paper texture, old age parchment feel. 
          Composition: Animal subject in center, background with star trails, astrological symbols, runes, or ruins. 
          Detailed line art, mysterious atmosphere.
          
          Subject: ${data.imageDescription}
        `;

        const response = await ai.models.generateContent({
          model: 'gemini-2.5-flash-image',
          contents: {
            parts: [{ text: stylePrompt }],
          },
          config: {
            imageConfig: {
              aspectRatio: "3:4"
            }
          }
        });

        // Parse response to find the image part
        let foundImage = false;
        if (response.candidates?.[0]?.content?.parts) {
            for (const part of response.candidates[0].content.parts) {
                if (part.inlineData) {
                    const base64EncodeString = part.inlineData.data;
                    const url = `data:image/png;base64,${base64EncodeString}`;
                    setImageUrl(url);
                    foundImage = true;
                    break;
                }
            }
        }

        if (!foundImage) {
             console.warn("No image found in response. Using fallback.");
             setImageUrl(`https://picsum.photos/seed/${data.id * 123}/800/1000`);
        }

      } catch (error) {
        console.error("Image generation failed:", error);
        // Fallback image on error
        setImageUrl(`https://picsum.photos/seed/${data.id * 123}/800/1000`);
      } finally {
        setLoading(false);
      }
    };

    generateImage();
  }, [data]);

  return (
    <div className="flex flex-col items-center justify-center min-h-[60vh] z-10 animate-fade-in w-full max-w-4xl mx-auto px-4 py-8">
      
      {/* Card Container - Changed to Portrait (3:4) Aspect Ratio for Tarot Feel */}
      <div className="relative w-full max-w-[340px] bg-slate-900 border-2 border-mystic-gold rounded-xl shadow-[0_0_50px_rgba(212,175,55,0.2)] overflow-hidden flex flex-col">
        
        {/* Decorative Corner borders */}
        <div className="absolute top-2 left-2 w-8 h-8 border-t-2 border-l-2 border-mystic-gold opacity-50 z-20"></div>
        <div className="absolute top-2 right-2 w-8 h-8 border-t-2 border-r-2 border-mystic-gold opacity-50 z-20"></div>
        <div className="absolute bottom-2 left-2 w-8 h-8 border-b-2 border-l-2 border-mystic-gold opacity-50 z-20"></div>
        <div className="absolute bottom-2 right-2 w-8 h-8 border-b-2 border-r-2 border-mystic-gold opacity-50 z-20"></div>

        {/* Header: Number & Quadrant */}
        <div className="p-4 text-center border-b border-mystic-gold/30 bg-gradient-to-b from-slate-800 to-slate-900 z-10">
            <div className="flex justify-between items-center px-2">
                <span className="text-xs uppercase tracking-[0.2em] text-slate-400">No. {data.id}</span>
                <span className="text-xs uppercase tracking-widest text-amber-500 font-bold">{data.quadrant.split(' ')[0]}...</span>
            </div>
        </div>

        {/* Image Area - Portrait 3:4 */}
        <div className="relative aspect-[3/4] w-full overflow-hidden bg-slate-950 group">
          {loading ? (
             /* Loading State */
             <div className="absolute inset-0 flex flex-col items-center justify-center bg-slate-900 text-mystic-gold gap-4">
                <div className="w-16 h-16 border-4 border-mystic-gold/30 border-t-mystic-gold rounded-full animate-spin"></div>
                <p className="text-xs tracking-widest uppercase animate-pulse">Manifesting Vision...</p>
             </div>
          ) : (
             /* Generated Image */
             <>
                <img 
                    src={imageUrl || ''} 
                    alt={data.arcanaName}
                    className="w-full h-full object-cover transition-transform duration-[2s] group-hover:scale-110"
                />
                {/* Vignette Overlay */}
                <div className="absolute inset-0 bg-gradient-to-t from-slate-900 via-transparent to-slate-900/40 pointer-events-none"></div>
             </>
          )}
        </div>

        {/* Content Body */}
        <div className="p-6 flex flex-col items-center text-center space-y-3 relative bg-slate-900 border-t border-mystic-gold/30">
           {/* Big Chinese Name */}
           <h1 className="text-4xl md:text-5xl font-serif text-mystic-gold font-bold mb-1 drop-shadow-lg">
             {data.chineseName}
           </h1>
           
           {/* Arcana Name */}
           <h2 className="text-sm md:text-base font-display text-mystic-gold-light tracking-wide uppercase border-b border-mystic-gold/30 pb-3 w-full">
             {data.arcanaName}
           </h2>

           {/* Divination Theme */}
           <div className="pt-2">
             <p className="text-[10px] text-slate-500 uppercase tracking-widest mb-1">Divination</p>
             <p className="text-lg font-serif text-white italic leading-relaxed">
               "{data.theme}"
             </p>
           </div>
        </div>

      </div>

      {/* Action Buttons */}
      <div className="mt-8 flex flex-col sm:flex-row gap-6">
        <button 
          onClick={onReset}
          className="px-8 py-3 bg-mystic-gold text-slate-900 font-display font-bold tracking-widest uppercase hover:bg-white hover:shadow-[0_0_20px_rgba(255,255,255,0.4)] transition-all duration-300 rounded-sm"
        >
          Draw Again
        </button>
      </div>

    </div>
  );
};

export default ResultCard;
import React, { useEffect, useState } from 'react';

const StarBackground: React.FC = () => {
  const [stars, setStars] = useState<{ id: number; left: number; top: number; delay: number; size: number }[]>([]);

  useEffect(() => {
    // Generate static stars to avoid re-rendering flickering
    const newStars = Array.from({ length: 50 }).map((_, i) => ({
      id: i,
      left: Math.random() * 100,
      top: Math.random() * 100,
      delay: Math.random() * 5,
      size: Math.random() * 3 + 1,
    }));
    setStars(newStars);
  }, []);

  return (
    <div className="fixed inset-0 pointer-events-none z-0 overflow-hidden">
      {/* Deep gradient background */}
      <div className="absolute inset-0 bg-gradient-to-br from-celestial-dark via-slate-900 to-deep-teal opacity-90"></div>
      
      {/* Generated Stars */}
      {stars.map((star) => (
        <div
          key={star.id}
          className="absolute rounded-full bg-white opacity-0 animate-twinkle"
          style={{
            left: `${star.left}%`,
            top: `${star.top}%`,
            width: `${star.size}px`,
            height: `${star.size}px`,
            animationDelay: `${star.delay}s`,
            boxShadow: `0 0 ${star.size * 2}px rgba(255, 255, 255, 0.8)`
          }}
        />
      ))}
      
      {/* Nebula/Fog Overlay */}
      <div className="absolute inset-0 bg-[url('https://www.transparenttextures.com/patterns/stardust.png')] opacity-20 mix-blend-overlay"></div>
    </div>
  );
};

export default StarBackground;
import { Constellation, Quadrant } from './types';

export const CONSTELLATIONS: Constellation[] = [
  // Azure Dragon (East)
  { 
    id: 1, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "角木蛟", 
    arcanaName: "The Jade Serpent of Arbor", 
    theme: "Renewal & Wisdom",
    imageDescription: "Serpentine Dragon of the Forest. A Wyvern-like or horned giant serpent coiled majestically around an ancient, gnarled tree in a mystical forest."
  },
  { 
    id: 2, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "亢金龙", 
    arcanaName: "The Gilded Dragon of Aura", 
    theme: "Authority & Triumph",
    imageDescription: "Gilded Western Dragon. A majestic dragon with immense wings and sharp, metallic scales shimmering with gold, perched atop a cathedral spire."
  },
  { 
    id: 3, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "氐土貉", 
    arcanaName: "The Earth Badger of Foundation", 
    theme: "Stability & Patience",
    imageDescription: "Chthonic Badger-Beast. A rugged, powerful badger-like creature with stone-like skin and crystalline horns, emerging from the deep earth roots."
  },
  { 
    id: 4, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "房日兔", 
    arcanaName: "The Solar Hare of Sanctuary", 
    theme: "Prosperity & Comfort",
    imageDescription: "Moonlit Mythic Hare. An elegant hare with fur patterned like star charts, sitting in a sanctuary of bioluminescent magical flora."
  },
  { 
    id: 5, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "心月狐", 
    arcanaName: "The Lunar Fox of Intuition", 
    theme: "Mystery & Insight",
    imageDescription: "Ethereal Lunar Fox-Spirit. A multi-tailed fox spirit with deep, glowing eyes, surrounded by floating spectral orbs under a crescent moon."
  },
  { 
    id: 6, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "尾火虎", 
    arcanaName: "The Fiery Tiger of Destiny", 
    theme: "Passion & Resolution",
    imageDescription: "Infernal Striped Tiger. A fierce tiger with coat patterns made of actual burning embers and flame, prowling through a volcanic obsidian landscape."
  },
  { 
    id: 7, 
    quadrant: Quadrant.AzureDragon, 
    chineseName: "箕水豹", 
    arcanaName: "The Water Leopard of Harvest", 
    theme: "Abundance & Voyage",
    imageDescription: "Abyssal Water Leopard. A streamlined, aquatic leopard with fins and scales, lurking gracefully amongst sunken marble ruins underwater."
  },
  
  // Black Tortoise (North)
  { 
    id: 8, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "斗木獬", 
    arcanaName: "The Judgement Ram of Aether", 
    theme: "Justice & Honor",
    imageDescription: "Horned Judgment Ram. A solemn ram with enormous, spiral horns that look like judge's gavels, standing upon a floating rock platform."
  },
  { 
    id: 9, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "牛金牛", 
    arcanaName: "The Golden Ox of Virgo", 
    theme: "Diligence & Wealth",
    imageDescription: "Celestial Golden Bull. A massive, sturdy ox made of bronze and gold, with constellations etched into its hide, carrying the weight of a sphere on its back."
  },
  { 
    id: 10, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "女土蝠", 
    arcanaName: "The Terran Bat of Chalice", 
    theme: "Receptivity & Shelter",
    imageDescription: "Gothic Cave Bat. A giant, terrifyingly beautiful bat with ornate, lace-like wings, hanging inside a crystal-encrusted cavern."
  },
  { 
    id: 11, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "虚日鼠", 
    arcanaName: "The Sun Rat of The Void", 
    theme: "Wasteland & Testing",
    imageDescription: "Shadow-Dweller Rat. A small but piercingly sharp rat, cloaked in shadows and purple smoke, sitting atop a pile of ancient skulls or relics."
  },
  { 
    id: 12, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "危月燕", 
    arcanaName: "The Lunar Swallow of Sorrow", 
    theme: "Warning & Reflection",
    imageDescription: "Ominous Mystic Swallow. A sleek swallow with dark, metallic feathers, flying against a backdrop of a gathering magical storm and lightning."
  },
  { 
    id: 13, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "室火猪", 
    arcanaName: "The Fire Boar of The Chamber", 
    theme: "Purification & Vigor",
    imageDescription: "Fiery Guardian Boar. A hulking boar with tusks of magma and a mane of fire, guarding the entrance to a secret stone chamber."
  },
  { 
    id: 14, 
    quadrant: Quadrant.BlackTortoise, 
    chineseName: "壁水貐", 
    arcanaName: "The Water Pig of The Veil", 
    theme: "Seclusion & Inner Peace",
    imageDescription: "Cryptic Water Beast. A mysterious, amphibious creature resembling a mythical hyena with scales, resting in a secluded, foggy lagoon."
  },

  // White Tiger (West)
  { 
    id: 15, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "奎木狼", 
    arcanaName: "The Timber Wolf of Cosmos", 
    theme: "Creation & Law",
    imageDescription: "Cosmic Timber Wolf. A regal wolf whose fur blends into the night sky, with galaxies swirling within its silhouette, howling at a clockwork moon."
  },
  { 
    id: 16, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "娄金狗", 
    arcanaName: "The Golden Hound of Pact", 
    theme: "Loyalty & Guardianship",
    imageDescription: "Loyal Golden Hound. A strong, armored hound with golden fur, sitting faithfully beside an ancient, locked treasure chest or gate."
  },
  { 
    id: 17, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "胃土雉", 
    arcanaName: "The Earth Pheasant of Satiety", 
    theme: "Completion & Satisfaction",
    imageDescription: "Abundant Earth Pheasant. An ornate pheasant with plumage resembling autumn leaves, surrounded by a cornucopia of magical fruits and harvest grains."
  },
  { 
    id: 18, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "昴日鸡", 
    arcanaName: "The Solar Fowl of The Pleiad", 
    theme: "Illumination & Vision",
    imageDescription: "Enlightened Solar Rooster. A proud rooster radiating intense beams of sunlight from its crown, standing on a high peak at dawn."
  },
  { 
    id: 19, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "毕月乌", 
    arcanaName: "The Lunar Crow of Harvest", 
    theme: "Observation & Discipline",
    imageDescription: "Observant Lunar Crow. A pitch-black crow with three eyes, perched on an ancient telescope or brass astrolabe, watching the stars."
  },
  { 
    id: 20, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "觜火猴", 
    arcanaName: "The Fiery Ape of The Muzzle", 
    theme: "Artifice & Trickery",
    imageDescription: "Mischievous Fiery Ape. A clever ape with burning markings on its face, holding a stolen magical artifact, grinning mischievously."
  },
  { 
    id: 21, 
    quadrant: Quadrant.WhiteTiger, 
    chineseName: "参水猿", 
    arcanaName: "The Water Ape of Orion", 
    theme: "Adventure & Discovery",
    imageDescription: "Adventurous Water Ape. A muscular ape with fur that flows like water, climbing a giant waterfall in a mythical jungle."
  },

  // Vermilion Bird (South)
  { 
    id: 22, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "井木犴", 
    arcanaName: "The Timber Jackal of The Well", 
    theme: "Exploration & Depth",
    imageDescription: "Ancient Timber Jackal. A jackal-like spirit with limbs made of roots and branches, peering down into a deep, glowing magical well."
  },
  { 
    id: 23, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "鬼金羊", 
    arcanaName: "The Golden Ram of The Phantom", 
    theme: "Transcendence & Loss",
    imageDescription: "Spectral Golden Ram. A translucent, ghostly ram glowing with faint gold light, wandering through a field of gravestones and mist."
  },
  { 
    id: 24, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "柳土獐", 
    arcanaName: "The Earth Roe of Salix", 
    theme: "Growth & Sustenance",
    imageDescription: "Fertile Willow Deer. A graceful deer with antlers made of weeping willow branches, standing in a lush, green glade."
  },
  { 
    id: 25, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "星日马", 
    arcanaName: "The Solar Steed of The Constellation", 
    theme: "Speed & Momentum",
    imageDescription: "Celestial Solar Steed. A galloping horse with a mane of pure solar fire, leaving a trail of stardust as it runs across the sky."
  },
  { 
    id: 26, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "张月鹿", 
    arcanaName: "The Lunar Deer of The Net", 
    theme: "Connection & Elegance",
    imageDescription: "Enchanted Lunar Deer. An elegant deer marked with a crescent moon symbol on its forehead, entangled gently in magical silver nets or vines."
  },
  { 
    id: 27, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "翼火蛇", 
    arcanaName: "The Fiery Serpent of The Aerie", 
    theme: "Protection & Retreat",
    imageDescription: "Winged Fiery Serpent. A serpent with feathered wings of fire, coiled protectively around a high mountain nest."
  },
  { 
    id: 28, 
    quadrant: Quadrant.VermilionBird, 
    chineseName: "轸水蚓", 
    arcanaName: "The Water Worm of The Chariot", 
    theme: "Movement & Progress",
    imageDescription: "Deepwater Sea Serpent. A colossal, worm-like sea serpent with bioluminescent scales, navigating the deepest, darkest ocean trench."
  },
];
