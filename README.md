import { useState, useEffect } from 'react';
import { AnimatePresence } from 'framer-motion';
import { Header } from '@/components/Header';
import { TopicMap } from '@/components/TopicMap';
import { MicroQuest } from '@/components/MicroQuest';
import { Topic, createTopic, updateTopicAfterQuest } from '@/lib/forgettingCurve';
import { Toaster } from '@/components/ui/sonner';
import { toast } from 'sonner';

// Sample topics for demo
const DEMO_TOPICS: Topic[] = [
  {
    id: 'demo-1',
    name: 'Photosynthesis',
    importance: 'high',
    lastStudied: Date.now() - 1000 * 60 * 60 * 48, // 48 hours ago
    strength: 18,
    correctRecalls: 2,
    totalAttempts: 3,
    createdAt: Date.now() - 1000 * 60 * 60 * 72,
  },
  {
    id: 'demo-2',
    name: 'French Revolution',
    importance: 'medium',
    lastStudied: Date.now() - 1000 * 60 * 60 * 8, // 8 hours ago
    strength: 12,
    correctRecalls: 1,
    totalAttempts: 1,
    createdAt: Date.now() - 1000 * 60 * 60 * 24,
  },
  {
    id: 'demo-3',
    name: 'Quadratic Equations',
    importance: 'high',
    lastStudied: Date.now() - 1000 * 60 * 60 * 72, // 72 hours ago
    strength: 10,
    correctRecalls: 0,
    totalAttempts: 1,
    createdAt: Date.now() - 1000 * 60 * 60 * 96,
  },
];

const Index = () => {
  const [topics, setTopics] = useState<Topic[]>([]);
  const [activeQuest, setActiveQuest] = useState<Topic | null>(null);
  
  // Load topics from localStorage or use demo data
  useEffect(() => {
    const saved = localStorage.getItem('knowledge-fog-topics');
    if (saved) {
      try {
        setTopics(JSON.parse(saved));
      } catch {
        setTopics(DEMO_TOPICS);
      }
    } else {
      setTopics(DEMO_TOPICS);
    }
  }, []);
  
  // Save topics to localStorage
  useEffect(() => {
    if (topics.length > 0) {
      localStorage.setItem('knowledge-fog-topics', JSON.stringify(topics));
    }
  }, [topics]);
  
  const handleAddTopic = (name: string, importance: 'low' | 'medium' | 'high') => {
    const newTopic = createTopic(name, importance);
    setTopics((prev) => [...prev, newTopic]);
    toast.success(`Added "${name}" to your knowledge map!`, {
      description: 'Start reviewing to keep the fog away.',
    });
  };
  
  const handleStartQuest = (topic: Topic) => {
    setActiveQuest(topic);
  };
  
  const handleQuestComplete = (isCorrect: boolean) => {
    if (!activeQuest) return;
    
    const updatedTopic = updateTopicAfterQuest(activeQuest, isCorrect);
    setTopics((prev) =>
      prev.map((t) => (t.id === updatedTopic.id ? updatedTopic : t))
    );
    
    if (isCorrect) {
      toast.success('Fog cleared! 🎉', {
        description: 'Your memory strength has increased.',
      });
    } else {
      toast.info('Keep practicing!', {
        description: 'Review this topic again soon.',
      });
    }
  };
  
  const handleCloseQuest = () => {
    setActiveQuest(null);
  };
  
  return (
    <div className="min-h-screen bg-background relative overflow-hidden">
      {/* Ambient background effects */}
      <div className="fixed inset-0 pointer-events-none">
        <div className="absolute top-1/4 -left-1/4 w-1/2 h-1/2 bg-primary/5 rounded-full blur-3xl" />
        <div className="absolute bottom-1/4 -right-1/4 w-1/2 h-1/2 bg-accent/10 rounded-full blur-3xl" />
        <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-full h-full bg-gradient-radial from-transparent via-transparent to-background" />
      </div>
      
      {/* Content */}
      <div className="relative z-10 container mx-auto px-4 py-6 max-w-6xl">
        <Header />
        
        <main>
          <TopicMap
            topics={topics}
            onAddTopic={handleAddTopic}
            onStartQuest={handleStartQuest}
          />
        </main>
        
        {/* Footer */}
        <footer className="text-center py-8 mt-12 text-sm text-muted-foreground">
          <p>Smart revision beats extra study. Clear the fog, retain the knowledge.</p>
        </footer>
      </div>
      
      {/* Micro Quest Modal */}
      <AnimatePresence>
        {activeQuest && (
          <MicroQuest
            topic={activeQuest}
            onComplete={handleQuestComplete}
            onClose={handleCloseQuest}
          />
        )}
      </AnimatePresence>
      
      <Toaster position="bottom-right" />
    </div>
  );
};

export default Index;
