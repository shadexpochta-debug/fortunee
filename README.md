import { useState, useEffect } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import confetti from "canvas-confetti";

const prizes = [25, 50, 100, 250];
const prizeGradients = [
  "from-yellow-400 to-yellow-600",
  "from-green-400 to-green-600",
  "from-blue-400 to-blue-600",
  "from-red-400 to-red-600",
];
const prizeMessages = {
  25: "Неплохо!",
  50: "Хороший результат!",
  100: "Отличный выигрыш!",
  250: "Джекпот!",
};

export default function FortuneWheel() {
  const [angle, setAngle] = useState(0);
  const [balance, setBalance] = useState(0);
  const [displayedBalance, setDisplayedBalance] = useState(0);
  const [spinning, setSpinning] = useState(false);
  const [won, setWon] = useState<number | null>(null);

  useEffect(() => {
    if (displayedBalance < balance) {
      const increment = Math.ceil((balance - displayedBalance) / 20);
      const timer = setTimeout(() => {
        setDisplayedBalance((prev) => Math.min(prev + increment, balance));
      }, 30);
      return () => clearTimeout(timer);
    }
  }, [displayedBalance, balance]);

  const launchConfetti = () => {
    confetti({
      particleCount: 100,
      spread: 70,
      origin: { y: 0.6 },
    });
  };

  const spinWheel = () => {
    if (spinning) return;

    setSpinning(true);
    setWon(null);

    const prizeIndex = Math.floor(Math.random() * prizes.length);
    const rotationPerSegment = 360 / prizes.length;
    const offset = Math.random() * rotationPerSegment;
    const spins = Math.floor(Math.random() * 3) + 5;
    const targetAngle = spins * 360 + prizeIndex * rotationPerSegment + offset;

    setAngle((prevAngle) => prevAngle + targetAngle);

    setTimeout(() => {
      const prize = prizes[prizeIndex];
      setBalance((b) => b + prize);
      setWon(prize);
      setSpinning(false);
      launchConfetti();
    }, 4000);
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] text-white p-4" style={{ fontFamily: 'Impact, sans-serif' }}>
      <h1 className="text-4xl md:text-6xl font-extrabold mb-6 tracking-tight text-center drop-shadow-lg animate-pulse">
        Колесо Фортуны
      </h1>

      <Card className="bg-zinc-900 p-6 rounded-2xl shadow-2xl mb-6 w-full max-w-sm">
        <CardContent className="flex flex-col items-center justify-center">
          <div className="text-lg font-semibold mb-2 text-zinc-400">Баланс</div>
          <motion.div
            key={displayedBalance}
            initial={{ scale: 0.8, opacity: 0.5 }}
            animate={{ scale: 1, opacity: 1 }}
            transition={{ duration: 0.3 }}
            className="text-4xl md:text-5xl font-extrabold text-green-400"
          >
            {displayedBalance} монет
          </motion.div>
        </CardContent>
      </Card>

      <div className="relative w-[320px] h-[320px] mb-8">
        <div className="absolute top-[-20px] left-1/2 z-20 transform -translate-x-1/2 rotate-180">
          <div className="w-0 h-0 border-l-[10px] border-r-[10px] border-b-[20px] border-l-transparent border-r-transparent border-b-yellow-400" />
        </div>

        <div className="w-full h-full rounded-full border-[10px] border-zinc-700 overflow-hidden relative shadow-2xl">
          <motion.div
            className="absolute w-full h-full origin-center"
            animate={{ rotate: angle }}
            transition={{ duration: 4, ease: "easeInOut" }}
          >
            {prizes.map((value, index) => {
              const rotation = 90 * index;
              const gradient = prizeGradients[index % prizeGradients.length];
              return (
                <div
                  key={index}
                  className="absolute w-full h-full origin-center"
                  style={{ transform: `rotate(${rotation}deg)` }}
                >
                  <div
                    className={`absolute top-0 left-1/2 w-1/2 h-1/2 origin-bottom-left bg-gradient-to-br ${gradient} rounded-bl-full`}
                    style={{
                      clipPath: "polygon(0% 0%, 100% 0%, 0% 100%)",
                      transform: "rotate(45deg) translate(-50%, 0)",
                    }}
                  >
                    <div className="w-full h-full flex items-center justify-center">
                      <div className="text-white text-2xl font-extrabold drop-shadow-md">
                        {value}
                      </div>
                    </div>
                  </div>
                </div>
              );
            })}
          </motion.div>
        </div>
      </div>

      <Button
        onClick={spinWheel}
        disabled={spinning}
        className="text-xl px-6 py-3 rounded-2xl font-bold shadow-lg bg-gradient-to-r from-green-400 to-green-600 hover:brightness-110 disabled:opacity-50"
      >
        {spinning ? "Крутим..." : "Крутить колесо"}
      </Button>

      <AnimatePresence>
        {won !== null && !spinning && (
          <motion.div
            className="mt-6 text-center"
            initial={{ opacity: 0, scale: 0.8 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.8 }}
            transition={{ duration: 0.5 }}
          >
            <div className="text-3xl font-extrabold text-yellow-400 drop-shadow-md">
              Вы выиграли {won} монет!
            </div>
            <div className="text-lg text-white mt-1 font-semibold animate-pulse">
              {prizeMessages[won]}
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}
