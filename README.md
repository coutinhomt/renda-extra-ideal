# renda-extra-ideal
Renda Extra Ideal é um Web App de diagnóstico de renda extra que utiliza análise de perfil para recomendar oportunidades personalizadas de geração de renda.

O sistema foi desenvolvido utilizando a plataforma Base44.

## Funcionalidades

- Diagnóstico de perfil de geração de renda
- Recomendação de oportunidades compatíveis
- Plano de ação inicial
- Estimativa de ganhos
- Assistente estratégico com IA

## Variáveis analisadas

- tempo disponível
- habilidades
- objetivos financeiros
- preferência online ou presencial
- investimento inicial
- disposição para aprender

## Resultado

O sistema gera recomendações personalizadas e um plano inicial de execução para ajudar o usuário a começar sua renda extra.

## Arquitetura

Frontend Web App  
Motor de diagnóstico baseado em regras de perfil  
Assistente estratégico com IA
import React, { useEffect, useRef } from "react";
import { Link, useNavigate, useLocation } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { LayoutDashboard, MessageCircle, TrendingUp, Wrench, ChevronLeft } from "lucide-react";
import { motion, AnimatePresence } from "framer-motion";

const navItems = [
  { label: "Início", page: "Dashboard", icon: LayoutDashboard },
  { label: "Assistente", page: "Assistant", icon: MessageCircle },
  { label: "Progresso", page: "Progress", icon: TrendingUp },
  { label: "Ferramentas", page: "Tools", icon: Wrench },
];

const PAGE_ORDER = ["Dashboard", "Assistant", "Progress", "Tools"];

export default function Layout({ children, currentPageName }) {
  const isQuiz = currentPageName === "Home";
  const isDashboard = currentPageName === "Dashboard";
  const showBack = !isQuiz && !isDashboard;
  const navigate = useNavigate();
  const location = useLocation();
  const prevPageRef = useRef(currentPageName);
  // Store last visited URL per tab
  const tabHistory = useRef({});

  useEffect(() => {
    const applyDark = (dark) => {
      document.documentElement.classList.toggle("dark", dark);
    };
    const mq = window.matchMedia("(prefers-color-scheme: dark)");
    applyDark(mq.matches);
    mq.addEventListener("change", (e) => applyDark(e.matches));
    return () => mq.removeEventListener("change", (e) => applyDark(e.matches));
  }, []);

  // Track last URL for each tab
  useEffect(() => {
    if (currentPageName && currentPageName !== "Home") {
      tabHistory.current[currentPageName] = location.pathname + location.search;
    }
  }, [location, currentPageName]);

  // Determine slide direction
  const prevIdx = PAGE_ORDER.indexOf(prevPageRef.current);
  const currIdx = PAGE_ORDER.indexOf(currentPageName);
  const dir = currIdx >= prevIdx ? 1 : -1;
  useEffect(() => { prevPageRef.current = currentPageName; }, [currentPageName]);

  const handleTabClick = (e, page) => {
    e.preventDefault();
    if (currentPageName === page) {
      // Reset: go to root of this tab
      tabHistory.current[page] = createPageUrl(page);
      navigate(createPageUrl(page), { replace: true });
    } else {
      // Restore last visited URL for this tab
      const dest = tabHistory.current[page] || createPageUrl(page);
      navigate(dest);
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-[#060d2e] via-[#0a1540] to-[#060d2e] text-white overflow-hidden">
      {!isQuiz && (
        <header
          className="sticky top-0 z-40 bg-[#060d2e]/90 backdrop-blur-xl border-b border-white/5 px-6 py-4"
          style={{ paddingTop: `calc(1rem + env(safe-area-inset-top))` }}
        >
          <div className="max-w-2xl mx-auto flex items-center justify-between">
            <div className="flex items-center gap-2">
              {showBack && (
                <button
                  onClick={() => navigate(-1)}
                  className="select-none w-8 h-8 flex items-center justify-center rounded-xl text-gray-400 hover:text-gray-200 hover:bg-white/5 transition-colors mr-1"
                >
                  <ChevronLeft className="w-5 h-5" />
                </button>
              )}
              <Link to={createPageUrl("Dashboard")} className="select-none font-bold text-lg bg-gradient-to-r from-amber-400 to-yellow-300 bg-clip-text text-transparent">
                Renda Extra
              </Link>
            </div>
            <nav className="hidden md:flex items-center gap-1">
              {navItems.map((item) => (
                <Link
                  key={item.page}
                  to={tabHistory.current[item.page] || createPageUrl(item.page)}
                  onClick={(e) => handleTabClick(e, item.page)}
                  className={`select-none flex items-center gap-2 px-4 py-2 rounded-xl text-sm font-medium transition-colors ${
                    currentPageName === item.page
                      ? "bg-amber-500/15 text-amber-400"
                      : "text-gray-400 hover:text-gray-200 hover:bg-white/5"
                  }`}
                >
                  <item.icon className="w-4 h-4" />
                  {item.label}
                </Link>
              ))}
            </nav>
          </div>
        </header>
      )}

      <AnimatePresence mode="wait" initial={false}>
        <motion.main
          key={currentPageName}
          initial={{ x: isQuiz ? 0 : `${dir * 100}%`, opacity: isQuiz ? 0 : 1 }}
          animate={{ x: 0, opacity: 1 }}
          exit={{ x: isQuiz ? 0 : `${-dir * 100}%`, opacity: isQuiz ? 0 : 1 }}
          transition={{ type: "spring", stiffness: 320, damping: 32, mass: 0.8 }}
          className={!isQuiz ? "pb-24 md:pb-8" : ""}
        >
          {children}
        </motion.main>
      </AnimatePresence>

      {!isQuiz && (
        <nav
          className="md:hidden fixed bottom-0 left-0 right-0 bg-[#060d2e]/95 backdrop-blur-xl border-t border-white/10 z-50"
          style={{ paddingBottom: `env(safe-area-inset-bottom)` }}
        >
          <div className="flex items-center justify-around py-2 px-2">
            {navItems.map((item) => {
              const isActive = currentPageName === item.page;
              return (
                <Link
                  key={item.page}
                  to={tabHistory.current[item.page] || createPageUrl(item.page)}
                  onClick={(e) => handleTabClick(e, item.page)}
                  className={`select-none flex flex-col items-center gap-1 px-3 py-2 rounded-xl transition-colors ${
                    isActive ? "text-amber-400" : "text-gray-500"
                  }`}
                >
                  <item.icon className="w-5 h-5" />
                  <span className="text-xs font-medium">{item.label}</span>
                </Link>
              );
            })}
          </div>
        </nav>
      )}
    </div>
  );
}
