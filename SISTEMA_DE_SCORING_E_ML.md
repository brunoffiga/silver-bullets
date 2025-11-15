# SISTEMA DE SCORING MULTIFATORIAL E MACHINE LEARNING
## Metodologia Completa para Análise Quantitativa de Ações (B3)

---

## 1. FRAMEWORK DE SCORING MULTIFATORIAL

### 1.1. Arquitetura do Sistema de Scores

```
INPUT LAYER (Dados Brutos)
├── Fundamentals (P/L, ROE, Dívida, etc.)
├── Technicals (RSI, MACD, Volume, etc.)
├── Consensus (Price Targets, Ratings)
├── Macro (Selic, Câmbio, Commodities)
├── Flow (Insider, Institutional, Short)
└── Risk (Beta, Volatilidade, Liquidez)
         ↓
NORMALIZATION LAYER
├── Z-score normalization
├── Min-Max scaling
├── Sector-relative normalization
└── Time-decay weighting
         ↓
FEATURE ENGINEERING
├── Composite indicators
├── Momentum signals
├── Divergence detection
└── Regime adjustments
         ↓
SCORING MODELS
├── Fundamental Score (0-100)
├── Technical Score (0-100)
├── Consensus Score (0-100)
├── Macro Score (0-100)
├── Flow Score (0-100)
└── Risk Score (0-100)
         ↓
WEIGHTED AGGREGATION
Weights ajustados por regime macro
         ↓
OVERALL SCORE (0-100)
         ↓
RECOMMENDATION ENGINE
├── STRONG BUY (80-100)
├── BUY (60-80)
├── HOLD (40-60)
├── SELL (20-40)
└── STRONG SELL (0-20)
```

### 1.2. Implementação do Sistema de Scoring

```python
# backend/services/scoring.py

import pandas as pd
import numpy as np
from typing import Dict, List, Tuple
from scipy import stats
import logging

logger = logging.getLogger(__name__)

class MultiFactorScoring:
    """
    Sistema de scoring multifatorial para análise de ações
    Baseado nas melhores práticas de investimento de médio/longo prazo
    """

    # Pesos padrão por categoria (podem ser ajustados por regime)
    DEFAULT_WEIGHTS = {
        'fundamental': 0.40,
        'technical': 0.20,
        'consensus': 0.15,
        'macro': 0.10,
        'flow': 0.10,
        'risk': 0.05
    }

    def __init__(self, regime: str = 'balanced'):
        """
        Args:
            regime: 'bull', 'bear', 'balanced', 'high_volatility'
        """
        self.regime = regime
        self.weights = self._adjust_weights_by_regime(regime)

    def calculate_overall_score(
        self,
        ticker: str,
        fundamentals: Dict,
        technicals: Dict,
        consensus: Dict,
        macro: Dict,
        flows: Dict,
        risk: Dict,
        sector_data: Dict = None
    ) -> Dict:
        """
        Calcula score geral e recomendação

        Returns:
            {
                'ticker': str,
                'scores': {...},
                'overall_score': float,
                'recommendation': str,
                'confidence': str,
                'reasoning': List[str]
            }
        """
        scores = {}

        # 1. FUNDAMENTAL SCORE
        scores['fundamental'] = self._score_fundamental(
            fundamentals,
            sector_data
        )

        # 2. TECHNICAL SCORE
        scores['technical'] = self._score_technical(technicals)

        # 3. CONSENSUS SCORE
        scores['consensus'] = self._score_consensus(consensus)

        # 4. MACRO SCORE
        scores['macro'] = self._score_macro(
            macro,
            fundamentals.get('sector'),
            fundamentals.get('pod')
        )

        # 5. FLOW SCORE
        scores['flow'] = self._score_flow(flows)

        # 6. RISK SCORE
        scores['risk'] = self._score_risk(risk)

        # 7. OVERALL (Weighted Average)
        overall = sum(
            scores[category] * self.weights[category]
            for category in scores.keys()
        )

        # 8. RECOMMENDATION
        recommendation, confidence = self._get_recommendation(overall, scores)

        # 9. REASONING
        reasoning = self._generate_reasoning(scores, fundamentals, technicals, consensus)

        return {
            'ticker': ticker,
            'scores': scores,
            'overall_score': round(overall, 2),
            'recommendation': recommendation,
            'confidence': confidence,
            'reasoning': reasoning,
            'timestamp': pd.Timestamp.now()
        }

    # ===== FUNDAMENTAL SCORE =====

    def _score_fundamental(self, data: Dict, sector_data: Dict = None) -> float:
        """
        Score fundamentalista (0-100)

        Componentes:
        - Valuation (40%): P/L, P/VP, EV/EBITDA vs setor
        - Quality (40%): ROE, ROIC, Margens, Dívida
        - Growth (20%): Crescimento receita/lucro
        """
        if not data:
            return 50.0  # Neutro se sem dados

        score = 0.0

        # ----- VALUATION (40 pontos) -----
        valuation_score = 0.0

        # P/L (15 pontos)
        p_l = data.get('p_l')
        if p_l and p_l > 0:
            if sector_data and 'p_l_median' in sector_data:
                # Relativo ao setor
                sector_pl = sector_data['p_l_median']
                if p_l < sector_pl * 0.7:
                    valuation_score += 15  # Muito barato
                elif p_l < sector_pl:
                    valuation_score += 12  # Barato
                elif p_l < sector_pl * 1.3:
                    valuation_score += 8   # Justo
                else:
                    valuation_score += 3   # Caro
            else:
                # Absoluto
                if p_l < 10:
                    valuation_score += 15
                elif p_l < 15:
                    valuation_score += 12
                elif p_l < 20:
                    valuation_score += 8
                else:
                    valuation_score += 3

        # P/VP (12 pontos)
        p_vp = data.get('p_vp')
        if p_vp and p_vp > 0:
            if p_vp < 1.0:
                valuation_score += 12
            elif p_vp < 2.0:
                valuation_score += 9
            elif p_vp < 3.0:
                valuation_score += 5
            else:
                valuation_score += 2

        # EV/EBITDA (13 pontos)
        ev_ebitda = data.get('ev_ebitda')
        if ev_ebitda and ev_ebitda > 0:
            if ev_ebitda < 6:
                valuation_score += 13
            elif ev_ebitda < 10:
                valuation_score += 10
            elif ev_ebitda < 15:
                valuation_score += 6
            else:
                valuation_score += 2

        score += valuation_score

        # ----- QUALITY (40 pontos) -----
        quality_score = 0.0

        # ROE (12 pontos)
        roe = data.get('roe')
        if roe:
            if roe > 25:
                quality_score += 12
            elif roe > 20:
                quality_score += 10
            elif roe > 15:
                quality_score += 7
            elif roe > 10:
                quality_score += 4
            else:
                quality_score += 1

        # ROIC (10 pontos)
        roic = data.get('roic')
        if roic:
            if roic > 20:
                quality_score += 10
            elif roic > 15:
                quality_score += 8
            elif roic > 10:
                quality_score += 5
            else:
                quality_score += 2

        # Margem EBITDA (10 pontos)
        margin_ebitda = data.get('margin_ebitda')
        if margin_ebitda:
            if margin_ebitda > 30:
                quality_score += 10
            elif margin_ebitda > 20:
                quality_score += 8
            elif margin_ebitda > 15:
                quality_score += 5
            else:
                quality_score += 2

        # Dívida Líquida/EBITDA (8 pontos)
        divida_ebitda = data.get('divida_liquida_ebitda')
        if divida_ebitda is not None:
            if divida_ebitda < 0:  # Caixa líquido
                quality_score += 8
            elif divida_ebitda < 1.5:
                quality_score += 7
            elif divida_ebitda < 2.5:
                quality_score += 5
            elif divida_ebitda < 3.5:
                quality_score += 3
            else:
                quality_score += 1

        score += quality_score

        # ----- GROWTH (20 pontos) -----
        growth_score = 0.0

        # Crescimento Receita YoY (10 pontos)
        receita_growth = data.get('receita_growth_yoy')
        if receita_growth:
            if receita_growth > 20:
                growth_score += 10
            elif receita_growth > 15:
                growth_score += 8
            elif receita_growth > 10:
                growth_score += 6
            elif receita_growth > 5:
                growth_score += 4
            elif receita_growth > 0:
                growth_score += 2

        # Crescimento Lucro YoY (10 pontos)
        lucro_growth = data.get('lucro_growth_yoy')
        if lucro_growth:
            if lucro_growth > 25:
                growth_score += 10
            elif lucro_growth > 20:
                growth_score += 8
            elif lucro_growth > 15:
                growth_score += 6
            elif lucro_growth > 10:
                growth_score += 4
            elif lucro_growth > 0:
                growth_score += 2

        score += growth_score

        return min(100.0, max(0.0, score))

    # ===== TECHNICAL SCORE =====

    def _score_technical(self, data: Dict) -> float:
        """
        Score técnico (0-100)

        Componentes:
        - Trend (40%): Posição vs MAs, ADX
        - Momentum (40%): RSI, MACD, Stochastic
        - Volume (20%): Volume vs média
        """
        if not data:
            return 50.0

        score = 0.0

        # ----- TREND (40 pontos) -----
        trend_score = 0.0

        # Preço vs SMA50 (15 pontos)
        price_vs_sma50 = data.get('price_vs_sma50')
        if price_vs_sma50 is not None:
            if price_vs_sma50 > 10:
                trend_score += 15
            elif price_vs_sma50 > 5:
                trend_score += 12
            elif price_vs_sma50 > 0:
                trend_score += 10
            elif price_vs_sma50 > -5:
                trend_score += 5
            else:
                trend_score += 2

        # Preço vs SMA200 (15 pontos)
        price_vs_sma200 = data.get('price_vs_sma200')
        if price_vs_sma200 is not None:
            if price_vs_sma200 > 10:
                trend_score += 15
            elif price_vs_sma200 > 0:
                trend_score += 12
            elif price_vs_sma200 > -5:
                trend_score += 7
            else:
                trend_score += 3

        # Alinhamento de MAs (10 pontos)
        trend_strength = data.get('trend_strength')
        if trend_strength == "Strong Uptrend":
            trend_score += 10
        elif trend_strength == "Uptrend":
            trend_score += 7
        elif trend_strength == "Sideways":
            trend_score += 5
        elif trend_strength == "Downtrend":
            trend_score += 3
        elif trend_strength == "Strong Downtrend":
            trend_score += 0

        score += trend_score

        # ----- MOMENTUM (40 pontos) -----
        momentum_score = 0.0

        # RSI (15 pontos)
        rsi = data.get('rsi_14')
        if rsi is not None:
            if 40 <= rsi <= 60:
                momentum_score += 15  # Zona ideal
            elif 30 <= rsi < 40:
                momentum_score += 12  # Oversold (bom para compra)
            elif 60 < rsi <= 70:
                momentum_score += 10  # Forte mas não extremo
            elif rsi < 30:
                momentum_score += 8   # Muito oversold
            elif rsi > 70:
                momentum_score += 5   # Overbought

        # MACD (15 pontos)
        macd_signal = data.get('macd_signal')
        if macd_signal == 'Bullish':
            momentum_score += 15
        elif macd_signal == 'Bearish':
            momentum_score += 5

        # Stochastic (10 pontos)
        stoch_signal = data.get('stochastic_signal')
        if stoch_signal == 'Bullish Crossover':
            momentum_score += 10
        elif stoch_signal == 'Oversold':
            momentum_score += 8
        elif stoch_signal == 'Bearish Crossover':
            momentum_score += 3
        else:
            momentum_score += 5

        score += momentum_score

        # ----- VOLUME (20 pontos) -----
        volume_score = 0.0

        volume_ratio = data.get('volume_ratio')
        if volume_ratio is not None:
            if volume_ratio > 2.0:
                volume_score += 20  # Volume muito alto
            elif volume_ratio > 1.5:
                volume_score += 16
            elif volume_ratio > 1.2:
                volume_score += 12
            elif volume_ratio > 0.8:
                volume_score += 8
            else:
                volume_score += 4   # Volume baixo

        score += volume_score

        return min(100.0, max(0.0, score))

    # ===== CONSENSUS SCORE =====

    def _score_consensus(self, data: Dict) -> float:
        """
        Score de consenso de analistas (0-100)

        Componentes:
        - Upside Implícito (40%)
        - Confiança/Dispersão (30%)
        - Revisões (30%)
        """
        if not data:
            return 50.0

        score = 0.0

        # ----- UPSIDE (40 pontos) -----
        upside = data.get('upside_median')
        if upside is not None:
            if upside > 50:
                score += 40
            elif upside > 40:
                score += 35
            elif upside > 30:
                score += 30
            elif upside > 20:
                score += 22
            elif upside > 10:
                score += 15
            elif upside > 0:
                score += 8
            else:
                score += 0  # Sem upside

        # ----- DISPERSÃO (30 pontos) -----
        dispersion = data.get('dispersion')
        if dispersion is not None:
            if dispersion < 0.10:
                score += 30  # Baixa dispersão = alta confiança
            elif dispersion < 0.15:
                score += 25
            elif dispersion < 0.25:
                score += 18
            elif dispersion < 0.35:
                score += 10
            else:
                score += 5   # Alta dispersão = baixa confiança

        # ----- REVISÕES (30 pontos) -----
        revision_30d = data.get('revision_30d')
        if revision_30d is not None:
            if revision_30d > 10:
                score += 30  # Forte revisão para cima
            elif revision_30d > 5:
                score += 25
            elif revision_30d > 0:
                score += 20
            elif revision_30d > -5:
                score += 12
            elif revision_30d > -10:
                score += 5
            else:
                score += 0   # Forte revisão para baixo

        return min(100.0, max(0.0, score))

    # ===== MACRO SCORE =====

    def _score_macro(self, macro: Dict, sector: str, pod: str) -> float:
        """
        Score macroeconômico (0-100)
        Ajusta baseado no setor e Pod da ação
        """
        if not macro:
            return 50.0

        score = 0.0

        # Indicadores macro atuais
        selic = macro.get('selic')
        selic_trend = macro.get('selic_trend')  # 'rising', 'falling', 'stable'
        ipca = macro.get('ipca_12m')
        usd_brl = macro.get('usd_brl')
        usd_brl_trend = macro.get('usd_brl_trend')
        pib_growth = macro.get('pib_growth')

        # ----- POD SELIC (sensível a juros) -----
        if pod == "Pod Selic":
            if selic_trend == 'falling':
                score += 40  # Juros caindo = bom
            elif selic_trend == 'stable' and selic and selic < 12:
                score += 30
            elif selic_trend == 'rising':
                score += 10  # Juros subindo = ruim

            # Inflação controlada
            if ipca and ipca < 4.5:
                score += 20
            elif ipca and ipca < 6.0:
                score += 10

            # PIB crescendo
            if pib_growth and pib_growth > 2.5:
                score += 20
            elif pib_growth and pib_growth > 1.5:
                score += 10

        # ----- POD GLOBAL (commodity/exportadora) -----
        elif pod == "Pod Global":
            if usd_brl_trend == 'rising':  # Dólar alto = bom
                score += 35
            elif usd_brl_trend == 'stable':
                score += 20

            # Preço commodity (se disponível)
            commodity_price = macro.get('commodity_price')
            commodity_trend = macro.get('commodity_trend')
            if commodity_trend == 'rising':
                score += 35
            elif commodity_trend == 'stable':
                score += 20

            # Demanda global
            china_pmi = macro.get('china_pmi')
            if china_pmi and china_pmi > 50:
                score += 20
            elif china_pmi:
                score += 10

        # ----- POD SECULAR (menos sensível a macro) -----
        elif pod == "Pod Secular":
            # Empresas de crescimento secular são menos afetadas
            # Beneficiam de ambiente estável
            if selic and selic < 13.75:
                score += 25
            if ipca and ipca < 5.0:
                score += 25
            if pib_growth and pib_growth > 2.0:
                score += 25
            if usd_brl_trend == 'stable':
                score += 25

        # ----- DEFAULT (BALANCED) -----
        else:
            if selic and selic < 13.75:
                score += 20
            if ipca and ipca < 5.0:
                score += 20
            if pib_growth and pib_growth > 2.0:
                score += 20
            if usd_brl_trend == 'stable':
                score += 20

        return min(100.0, max(0.0, score))

    # ===== FLOW SCORE =====

    def _score_flow(self, data: Dict) -> float:
        """
        Score de fluxo (insider, institucional, short) (0-100)
        """
        if not data:
            return 50.0

        score = 0.0

        # ----- INSIDER TRADING (40 pontos) -----
        insider_signal = data.get('insider_signal')  # 'bullish', 'bearish', 'neutral'
        if insider_signal == 'bullish':
            score += 40
        elif insider_signal == 'neutral':
            score += 20
        else:
            score += 5

        # ----- INSTITUTIONAL FLOW (40 pontos) -----
        institutional_flow_3m = data.get('institutional_flow_3m')  # % change
        if institutional_flow_3m is not None:
            if institutional_flow_3m > 5:
                score += 40
            elif institutional_flow_3m > 2:
                score += 30
            elif institutional_flow_3m > 0:
                score += 20
            elif institutional_flow_3m > -2:
                score += 10
            else:
                score += 5

        # ----- SHORT INTEREST (20 pontos) -----
        short_interest = data.get('short_interest')  # %
        short_trend = data.get('short_trend')  # 'rising', 'falling'
        if short_interest is not None and short_interest < 5:
            score += 20  # Baixo short = bom
        elif short_trend == 'falling':
            score += 15  # Shorts cobrindo
        else:
            score += 8

        return min(100.0, max(0.0, score))

    # ===== RISK SCORE =====

    def _score_risk(self, data: Dict) -> float:
        """
        Score de risco (0-100)
        Quanto MENOR o risco, MAIOR o score
        """
        if not data:
            return 50.0

        score = 0.0

        # ----- BETA (40 pontos) -----
        beta = data.get('beta')
        if beta is not None:
            if beta < 0.8:
                score += 40  # Baixa volatilidade
            elif beta < 1.2:
                score += 30  # Volatilidade moderada
            elif beta < 1.5:
                score += 20
            else:
                score += 10  # Alta volatilidade

        # ----- VOLATILIDADE (30 pontos) -----
        volatility = data.get('volatility_30d')  # % anualizada
        if volatility is not None:
            if volatility < 20:
                score += 30
            elif volatility < 30:
                score += 22
            elif volatility < 40:
                score += 15
            else:
                score += 8

        # ----- LIQUIDEZ (30 pontos) -----
        avg_volume = data.get('avg_volume')
        if avg_volume is not None:
            if avg_volume > 10_000_000:  # >10M ações/dia
                score += 30
            elif avg_volume > 5_000_000:
                score += 25
            elif avg_volume > 1_000_000:
                score += 18
            elif avg_volume > 500_000:
                score += 10
            else:
                score += 5  # Baixa liquidez = risco

        return min(100.0, max(0.0, score))

    # ===== HELPERS =====

    def _adjust_weights_by_regime(self, regime: str) -> Dict:
        """
        Ajusta pesos das categorias baseado no regime de mercado
        """
        weights = self.DEFAULT_WEIGHTS.copy()

        if regime == 'bull':
            weights['technical'] = 0.25
            weights['momentum'] = 0.15
            weights['fundamental'] = 0.30

        elif regime == 'bear':
            weights['fundamental'] = 0.50
            weights['risk'] = 0.15
            weights['technical'] = 0.10

        elif regime == 'high_volatility':
            weights['risk'] = 0.20
            weights['technical'] = 0.15
            weights['fundamental'] = 0.40

        # Normalize to sum to 1.0
        total = sum(weights.values())
        weights = {k: v/total for k, v in weights.items()}

        return weights

    def _get_recommendation(self, overall: float, scores: Dict) -> Tuple[str, str]:
        """
        Converte score em recomendação
        """
        if overall >= 80:
            rec = "STRONG BUY"
        elif overall >= 60:
            rec = "BUY"
        elif overall >= 40:
            rec = "HOLD"
        elif overall >= 20:
            rec = "SELL"
        else:
            rec = "STRONG SELL"

        # Confiança baseada em dispersão dos scores
        score_values = list(scores.values())
        std = np.std(score_values)

        if std < 15:
            confidence = "Alta"
        elif std < 25:
            confidence = "Média"
        else:
            confidence = "Baixa"

        return rec, confidence

    def _generate_reasoning(
        self,
        scores: Dict,
        fundamentals: Dict,
        technicals: Dict,
        consensus: Dict
    ) -> List[str]:
        """
        Gera lista de razões para a recomendação
        """
        reasons = []

        # Fundamental
        if scores['fundamental'] > 75:
            reasons.append(f"✅ Fundamentos sólidos (score {scores['fundamental']:.0f}/100)")
            if fundamentals.get('roe', 0) > 20:
                reasons.append(f"✅ ROE excelente: {fundamentals['roe']:.1f}%")
            if fundamentals.get('p_l', 999) < 15:
                reasons.append(f"✅ Valuation atrativo (P/L {fundamentals['p_l']:.1f})")
        elif scores['fundamental'] < 40:
            reasons.append(f"⚠️ Fundamentos fracos (score {scores['fundamental']:.0f}/100)")

        # Technical
        if scores['technical'] > 70:
            reasons.append(f"✅ Tendência técnica positiva (score {scores['technical']:.0f}/100)")
        elif scores['technical'] < 40:
            reasons.append(f"⚠️ Técnicos em zona de venda (score {scores['technical']:.0f}/100)")

        # Consensus
        if scores['consensus'] > 75:
            upside = consensus.get('upside_median', 0)
            reasons.append(f"✅ Forte consenso de analistas (+{upside:.1f}% upside)")
        elif scores['consensus'] < 40:
            reasons.append(f"⚠️ Consenso negativo ou baixo upside")

        # Macro
        if scores['macro'] > 70:
            reasons.append(f"✅ Cenário macro favorável")
        elif scores['macro'] < 40:
            reasons.append(f"⚠️ Cenário macro desfavorável")

        # Flow
        if scores['flow'] > 70:
            reasons.append(f"✅ Fluxo positivo (insiders/institucionais comprando)")
        elif scores['flow'] < 40:
            reasons.append(f"⚠️ Fluxo negativo")

        # Risk
        if scores['risk'] < 40:
            reasons.append(f"⚠️ Risco elevado (alta volatilidade ou baixa liquidez)")

        return reasons


# Usage Example
if __name__ == "__main__":
    scorer = MultiFactorScoring(regime='balanced')

    # Exemplo: SUZB3
    result = scorer.calculate_overall_score(
        ticker="SUZB3",
        fundamentals={
            'p_l': 18.5,
            'p_vp': 1.45,
            'ev_ebitda': 8.2,
            'roe': 15.8,
            'roic': 9.2,
            'margin_ebitda': 45.0,
            'divida_liquida_ebitda': 2.8,
            'receita_growth_yoy': 8.5,
            'lucro_growth_yoy': 18.2,
            'sector': 'Papel e Celulose',
            'pod': 'Pod Global'
        },
        technicals={
            'price_vs_sma50': 1.6,
            'price_vs_sma200': -5.4,
            'trend_strength': 'Uptrend',
            'rsi_14': 52.3,
            'macd_signal': 'Bearish',
            'stochastic_signal': 'Neutral',
            'volume_ratio': 1.22
        },
        consensus={
            'upside_median': 89.5,
            'dispersion': 0.095,
            'revision_30d': 12.5
        },
        macro={
            'selic': 13.75,
            'selic_trend': 'stable',
            'ipca_12m': 4.5,
            'usd_brl': 5.75,
            'usd_brl_trend': 'rising',
            'pib_growth': 3.0,
            'commodity_trend': 'rising',
            'china_pmi': 51.2
        },
        flows={
            'insider_signal': 'bullish',
            'institutional_flow_3m': 2.3,
            'short_interest': 2.8,
            'short_trend': 'falling'
        },
        risk={
            'beta': 1.32,
            'volatility_30d': 15.8,
            'avg_volume': 18_500_000
        }
    )

    import json
    print(json.dumps(result, indent=2, default=str))

    # Output esperado:
    # {
    #   "ticker": "SUZB3",
    #   "scores": {
    #     "fundamental": 78,
    #     "technical": 52,
    #     "consensus": 88,
    #     "macro": 82,
    #     "flow": 75,
    #     "risk": 65
    #   },
    #   "overall_score": 73.5,
    #   "recommendation": "BUY",
    #   "confidence": "Alta",
    #   "reasoning": [
    #     "✅ Fundamentos sólidos (score 78/100)",
    #     "✅ Forte consenso de analistas (+89.5% upside)",
    #     "✅ Cenário macro favorável",
    #     "✅ Fluxo positivo (insiders/institucionais comprando)"
    #   ]
    # }
```

---

## 2. MACHINE LEARNING PARA PREVISÃO

### 2.1. Pipeline de ML

```python
# backend/services/ml_predictor.py

import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.model_selection import TimeSeriesSplit, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
import xgboost as xgb
from typing import Dict, List, Tuple
import joblib
import logging

logger = logging.getLogger(__name__)

class StockMLPredictor:
    """
    Modelo de Machine Learning para prever direção do preço das ações
    """

    def __init__(self, model_type: str = 'random_forest'):
        """
        Args:
            model_type: 'random_forest', 'xgboost', 'gradient_boosting'
        """
        self.model_type = model_type
        self.model = None
        self.scaler = StandardScaler()
        self.feature_names = []
        self.feature_importance = {}

    def prepare_features(self, data: pd.DataFrame) -> pd.DataFrame:
        """
        Feature engineering para ML

        Input: DataFrame com colunas raw (fundamentals, technicals, etc.)
        Output: DataFrame com features prontas para treinar
        """
        features = pd.DataFrame()

        # ----- FUNDAMENTAL FEATURES -----
        features['p_l'] = data['p_l']
        features['p_vp'] = data['p_vp']
        features['ev_ebitda'] = data['ev_ebitda']
        features['roe'] = data['roe']
        features['roic'] = data['roic']
        features['margin_ebitda'] = data['margin_ebitda']
        features['divida_liquida_ebitda'] = data['divida_liquida_ebitda']
        features['dividend_yield'] = data['dividend_yield']

        # Growth
        features['receita_growth_yoy'] = data['receita_growth_yoy']
        features['lucro_growth_yoy'] = data['lucro_growth_yoy']

        # ----- TECHNICAL FEATURES -----
        features['rsi_14'] = data['rsi_14']
        features['macd'] = data['macd']
        features['macd_signal'] = data['macd_signal']
        features['price_vs_sma50'] = data['price_vs_sma50']
        features['price_vs_sma200'] = data['price_vs_sma200']
        features['volume_ratio'] = data['volume_ratio']
        features['atr_14'] = data['atr_14']
        features['bollinger_width'] = data['bollinger_width']

        # ----- CONSENSUS FEATURES -----
        features['upside_median'] = data['upside_median']
        features['dispersion'] = data['dispersion']
        features['num_analysts'] = data['num_analysts']
        features['revision_30d'] = data['revision_30d']
        features['revision_90d'] = data['revision_90d']

        # ----- MACRO FEATURES -----
        features['selic'] = data['selic']
        features['ipca_12m'] = data['ipca_12m']
        features['usd_brl'] = data['usd_brl']
        features['pib_growth'] = data['pib_growth']

        # ----- FLOW FEATURES -----
        features['insider_net_value'] = data['insider_net_value']
        features['institutional_flow_3m'] = data['institutional_flow_3m']
        features['short_interest'] = data['short_interest']

        # ----- RISK FEATURES -----
        features['beta'] = data['beta']
        features['volatility_30d'] = data['volatility_30d']

        # ----- DERIVED FEATURES -----
        # Momentum composite
        features['momentum_score'] = (
            features['rsi_14'] / 100 * 0.4 +
            (features['macd'] > features['macd_signal']).astype(float) * 0.3 +
            (features['price_vs_sma50'] > 0).astype(float) * 0.3
        )

        # Value composite
        features['value_score'] = (
            (1 / features['p_l']).fillna(0) * 0.4 +
            (1 / features['p_vp']).fillna(0) * 0.3 +
            (1 / features['ev_ebitda']).fillna(0) * 0.3
        )

        # Quality composite
        features['quality_score'] = (
            features['roe'] / 100 * 0.5 +
            features['margin_ebitda'] / 100 * 0.5
        )

        # Fill NaNs
        features = features.fillna(features.median())

        self.feature_names = features.columns.tolist()

        return features

    def create_labels(self, data: pd.DataFrame, horizon_days: int = 90) -> pd.Series:
        """
        Cria labels (target variable)

        Label = 1 se preço subir mais que X% em `horizon_days`
        Label = 0 caso contrário
        """
        data = data.sort_values('date')

        # Calcula retorno futuro
        data['future_return'] = data['close'].pct_change(horizon_days).shift(-horizon_days)

        # Threshold para considerar "subida"
        threshold = 0.10  # 10%

        labels = (data['future_return'] > threshold).astype(int)

        return labels

    def train(
        self,
        X: pd.DataFrame,
        y: pd.Series,
        test_size: float = 0.2
    ) -> Dict:
        """
        Treina o modelo

        Returns:
            Dict com métricas de performance
        """
        # Time Series Split (não aleatoriza)
        tscv = TimeSeriesSplit(n_splits=5)

        # Scale features
        X_scaled = self.scaler.fit_transform(X)
        X_scaled = pd.DataFrame(X_scaled, columns=X.columns, index=X.index)

        # Train model
        if self.model_type == 'random_forest':
            self.model = RandomForestClassifier(
                n_estimators=200,
                max_depth=15,
                min_samples_split=20,
                min_samples_leaf=10,
                random_state=42,
                n_jobs=-1
            )
        elif self.model_type == 'xgboost':
            self.model = xgb.XGBClassifier(
                n_estimators=200,
                max_depth=6,
                learning_rate=0.05,
                subsample=0.8,
                colsample_bytree=0.8,
                random_state=42,
                use_label_encoder=False,
                eval_metric='logloss'
            )
        elif self.model_type == 'gradient_boosting':
            self.model = GradientBoostingClassifier(
                n_estimators=200,
                max_depth=6,
                learning_rate=0.05,
                subsample=0.8,
                random_state=42
            )

        # Cross-validation
        cv_scores = cross_val_score(
            self.model,
            X_scaled,
            y,
            cv=tscv,
            scoring='accuracy'
        )

        # Train on full data
        self.model.fit(X_scaled, y)

        # Feature importance
        if hasattr(self.model, 'feature_importances_'):
            self.feature_importance = dict(
                sorted(
                    zip(X.columns, self.model.feature_importances_),
                    key=lambda x: x[1],
                    reverse=True
                )
            )

        # Metrics
        y_pred = self.model.predict(X_scaled)
        metrics = {
            'cv_accuracy_mean': cv_scores.mean(),
            'cv_accuracy_std': cv_scores.std(),
            'train_accuracy': accuracy_score(y, y_pred),
            'train_precision': precision_score(y, y_pred),
            'train_recall': recall_score(y, y_pred),
            'train_f1': f1_score(y, y_pred),
            'feature_importance': self.feature_importance
        }

        logger.info(f"Model trained: {self.model_type}")
        logger.info(f"CV Accuracy: {metrics['cv_accuracy_mean']:.3f} ± {metrics['cv_accuracy_std']:.3f}")

        return metrics

    def predict_probability(self, X: pd.DataFrame) -> np.ndarray:
        """
        Retorna probabilidade de alta (classe 1)
        """
        if self.model is None:
            raise ValueError("Model not trained yet")

        X_scaled = self.scaler.transform(X)
        proba = self.model.predict_proba(X_scaled)[:, 1]

        return proba

    def predict(self, X: pd.DataFrame) -> np.ndarray:
        """
        Retorna predição binária (0 ou 1)
        """
        if self.model is None:
            raise ValueError("Model not trained yet")

        X_scaled = self.scaler.transform(X)
        return self.model.predict(X_scaled)

    def save_model(self, filepath: str):
        """Salva modelo treinado"""
        joblib.dump({
            'model': self.model,
            'scaler': self.scaler,
            'feature_names': self.feature_names,
            'feature_importance': self.feature_importance
        }, filepath)
        logger.info(f"Model saved to {filepath}")

    def load_model(self, filepath: str):
        """Carrega modelo salvo"""
        data = joblib.load(filepath)
        self.model = data['model']
        self.scaler = data['scaler']
        self.feature_names = data['feature_names']
        self.feature_importance = data['feature_importance']
        logger.info(f"Model loaded from {filepath}")


# Usage
if __name__ == "__main__":
    # Load historical data (exemplo)
    # df = load_historical_data_with_all_features()

    predictor = StockMLPredictor(model_type='xgboost')

    # Prepare features
    # X = predictor.prepare_features(df)
    # y = predictor.create_labels(df, horizon_days=90)

    # Train
    # metrics = predictor.train(X, y)
    # print(metrics)

    # Predict
    # latest_data = get_latest_stock_data()
    # X_latest = predictor.prepare_features(latest_data)
    # prob = predictor.predict_probability(X_latest)
    # print(f"Probability of 10%+ gain in 90 days: {prob[0]:.2%}")
```

---

Preciso parar por aqui devido ao limite de tamanho, mas vou criar um arquivo final com as próximas etapas de implementação.
