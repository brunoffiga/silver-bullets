# PLANO DE IMPLEMENTAÇÃO TÉCNICO
## Roadmap para Dashboard de Análise de Ações (B3) - Nível Profissional

---

## ARQUITETURA PROPOSTA

### Stack Tecnológico Recomendado:

```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND (UI)                          │
│  • HTML/CSS/JavaScript (atual)                              │
│  • Chart.js (gráficos) ✅                                    │
│  • + DataTables.js (tabelas avançadas)                      │
│  • + ApexCharts (gráficos financeiros profissionais)        │
│  • + TradingView Widgets (opcional)                         │
└─────────────────────────────────────────────────────────────┘
                              ↓ API REST
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND (API Server)                     │
│  • Python Flask/FastAPI (recomendado)                       │
│  • Node.js + Express (alternativa)                          │
│                                                              │
│  Funções:                                                    │
│  - Servir dados processados                                 │
│  - Cache inteligente (Redis)                                │
│  - Rate limiting                                             │
│  - Autenticação (se necessário)                             │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                  DATA PROCESSING LAYER                      │
│  • Python (Pandas, NumPy)                                   │
│  • TA-Lib (indicadores técnicos)                            │
│  • Scikit-learn (ML)                                        │
│                                                              │
│  Funções:                                                    │
│  - Cálculo de indicadores                                   │
│  - Normalização de dados                                    │
│  - Scoring models                                            │
│  - Backtesting                                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   DATA COLLECTION LAYER                     │
│  • Python (Requests, BeautifulSoup, Selenium)               │
│  • Scrapers para bancos/portais                             │
│  • Integrações API                                           │
│                                                              │
│  Schedulers:                                                 │
│  - Celery (tasks assíncronas)                               │
│  - Cron jobs                                                 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      DATA SOURCES                           │
│  APIs:                                                       │
│  • Yahoo Finance (preços)                                    │
│  • Banco Central (Selic, IPCA, Câmbio)                      │
│  • Alpha Vantage (dados financeiros)                        │
│  • Status Invest (fundamentos B3)                           │
│                                                              │
│  Scraping:                                                   │
│  • XP, BTG, Itaú (price targets)                            │
│  • CVM (insider trading)                                     │
│  • B3 (dados corporativos)                                   │
│  • Portais de notícias                                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        DATABASE                             │
│  • PostgreSQL (dados estruturados)                          │
│  • TimescaleDB (séries temporais - preços)                  │
│  • Redis (cache)                                             │
│                                                              │
│  Schemas:                                                    │
│  - companies (dados cadastrais)                             │
│  - prices (histórico de preços)                             │
│  - fundamentals (indicadores fundamentalistas)              │
│  - consensus (price targets, ratings)                       │
│  - macro (indicadores macroeconômicos)                      │
│  - events (notícias, earnings, etc.)                        │
└─────────────────────────────────────────────────────────────┘
```

---

## FASE 1: FUNDAÇÃO (Semanas 1-2)

### 1.1. Setup do Ambiente de Desenvolvimento

**Criar estrutura de projeto:**

```
silver-bullets/
├── backend/
│   ├── app.py                    # Flask/FastAPI app
│   ├── config.py                 # Configurações
│   ├── requirements.txt
│   ├── models/
│   │   ├── database.py           # ORM models
│   │   └── schemas.py            # Pydantic schemas
│   ├── services/
│   │   ├── data_fetcher.py       # Coleta de dados
│   │   ├── indicators.py         # Cálculo de indicadores
│   │   ├── scoring.py            # Sistema de scoring
│   │   └── backtesting.py
│   ├── scrapers/
│   │   ├── yahoo_finance.py
│   │   ├── status_invest.py
│   │   ├── fundamentus.py
│   │   └── analyst_scraper.py    # XP, BTG, etc.
│   └── utils/
│       ├── cache.py
│       └── helpers.py
├── frontend/
│   ├── index.html                # Dashboard principal
│   ├── css/
│   │   └── styles.css
│   ├── js/
│   │   ├── main.js
│   │   ├── api.js                # Chamadas à API
│   │   ├── charts.js
│   │   └── components/
│   │       ├── stockTable.js
│   │       ├── simulator.js
│   │       └── filters.js
│   └── assets/
├── data/
│   ├── raw/                      # Dados brutos
│   ├── processed/                # Dados processados
│   └── cache/
├── database/
│   ├── migrations/
│   └── seeds/                    # Dados iniciais
├── tests/
│   ├── test_indicators.py
│   ├── test_scoring.py
│   └── test_api.py
├── docs/
│   ├── ANALISE_CRITICA_E_MELHORIAS.md
│   ├── PLANO_DE_IMPLEMENTACAO_TECNICO.md
│   └── API_DOCUMENTATION.md
├── .env                          # Variáveis de ambiente
├── docker-compose.yml            # PostgreSQL, Redis
└── README.md
```

**requirements.txt (Backend Python):**

```txt
# Web Framework
fastapi==0.104.1
uvicorn==0.24.0
python-multipart==0.0.6

# Database
psycopg2-binary==2.9.9
sqlalchemy==2.0.23
alembic==1.12.1

# Data Processing
pandas==2.1.3
numpy==1.26.2
scipy==1.11.4

# Technical Analysis
TA-Lib==0.4.28
pandas-ta==0.3.14b

# Machine Learning
scikit-learn==1.3.2
xgboost==2.0.2

# Data Fetching
requests==2.31.0
beautifulsoup4==4.12.2
selenium==4.15.2
yfinance==0.2.32

# Cache & Tasks
redis==5.0.1
celery==5.3.4

# Utils
python-dotenv==1.0.0
pydantic==2.5.2
pydantic-settings==2.1.0
```

### 1.2. Database Schema (PostgreSQL)

```sql
-- companies table
CREATE TABLE companies (
    id SERIAL PRIMARY KEY,
    ticker VARCHAR(10) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    sector VARCHAR(100),
    subsector VARCHAR(100),
    pod VARCHAR(50),  -- Pod Secular, Pod Global, Pod Selic
    listing_segment VARCHAR(50),  -- Novo Mercado, etc.
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- prices table (TimescaleDB hypertable)
CREATE TABLE prices (
    ticker VARCHAR(10) NOT NULL,
    date DATE NOT NULL,
    open DECIMAL(12, 2),
    high DECIMAL(12, 2),
    low DECIMAL(12, 2),
    close DECIMAL(12, 2),
    volume BIGINT,
    adjusted_close DECIMAL(12, 2),
    PRIMARY KEY (ticker, date)
);
SELECT create_hypertable('prices', 'date');

-- fundamentals table
CREATE TABLE fundamentals (
    id SERIAL PRIMARY KEY,
    ticker VARCHAR(10) NOT NULL,
    report_date DATE NOT NULL,

    -- Valuation
    market_cap DECIMAL(18, 2),
    enterprise_value DECIMAL(18, 2),
    p_l DECIMAL(8, 2),
    p_vp DECIMAL(8, 2),
    ev_ebitda DECIMAL(8, 2),
    psr DECIMAL(8, 2),
    peg_ratio DECIMAL(8, 2),

    -- Profitability
    roe DECIMAL(8, 2),
    roic DECIMAL(8, 2),
    roa DECIMAL(8, 2),
    margin_bruta DECIMAL(8, 2),
    margin_ebitda DECIMAL(8, 2),
    margin_liquida DECIMAL(8, 2),

    -- Financial Health
    liquidez_corrente DECIMAL(8, 2),
    divida_liquida DECIMAL(18, 2),
    divida_liquida_ebitda DECIMAL(8, 2),
    divida_liquida_patrimonio DECIMAL(8, 2),

    -- Growth
    receita_12m DECIMAL(18, 2),
    receita_growth_yoy DECIMAL(8, 2),
    lucro_12m DECIMAL(18, 2),
    lucro_growth_yoy DECIMAL(8, 2),

    -- Dividends
    dividend_yield DECIMAL(8, 2),
    payout_ratio DECIMAL(8, 2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(ticker, report_date)
);

-- consensus table (price targets)
CREATE TABLE consensus (
    id SERIAL PRIMARY KEY,
    ticker VARCHAR(10) NOT NULL,
    firm VARCHAR(100) NOT NULL,  -- XP, BTG, Itaú, etc.
    analyst_name VARCHAR(255),

    target_price DECIMAL(12, 2),
    recommendation VARCHAR(20),  -- STRONG BUY, BUY, HOLD, SELL

    report_date DATE NOT NULL,
    horizon_months INT DEFAULT 12,  -- Horizonte do target (12m, 24m)

    -- Forward estimates
    eps_estimate_current_year DECIMAL(8, 2),
    eps_estimate_next_year DECIMAL(8, 2),
    revenue_estimate_current_year DECIMAL(18, 2),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(ticker, firm, report_date)
);

-- insider_trading table
CREATE TABLE insider_trading (
    id SERIAL PRIMARY KEY,
    ticker VARCHAR(10) NOT NULL,
    insider_name VARCHAR(255),
    insider_role VARCHAR(100),  -- CEO, CFO, Director, etc.

    transaction_type VARCHAR(10),  -- BUY, SELL
    shares BIGINT,
    price_per_share DECIMAL(12, 2),
    total_value DECIMAL(18, 2),

    transaction_date DATE NOT NULL,
    filing_date DATE,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- macro_indicators table
CREATE TABLE macro_indicators (
    id SERIAL PRIMARY KEY,
    indicator VARCHAR(50) NOT NULL,  -- SELIC, IPCA, USD_BRL, etc.
    date DATE NOT NULL,
    value DECIMAL(12, 4),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(indicator, date)
);

-- events table (earnings, dividends, news)
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    ticker VARCHAR(10),
    event_type VARCHAR(50) NOT NULL,  -- EARNINGS, DIVIDEND, NEWS, M&A

    title TEXT,
    description TEXT,
    url VARCHAR(500),

    event_date DATE NOT NULL,

    -- Earnings specific
    eps_actual DECIMAL(8, 2),
    eps_estimate DECIMAL(8, 2),
    eps_surprise_pct DECIMAL(8, 2),

    -- Dividend specific
    dividend_value DECIMAL(8, 2),
    ex_date DATE,
    pay_date DATE,

    -- Sentiment
    sentiment_score DECIMAL(4, 2),  -- -1 to 1

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- technical_indicators table
CREATE TABLE technical_indicators (
    ticker VARCHAR(10) NOT NULL,
    date DATE NOT NULL,

    -- Trend
    sma_20 DECIMAL(12, 2),
    sma_50 DECIMAL(12, 2),
    sma_200 DECIMAL(12, 2),
    ema_20 DECIMAL(12, 2),

    -- Momentum
    rsi_14 DECIMAL(8, 2),
    macd DECIMAL(12, 4),
    macd_signal DECIMAL(12, 4),
    macd_histogram DECIMAL(12, 4),
    stochastic_k DECIMAL(8, 2),
    stochastic_d DECIMAL(8, 2),

    -- Volatility
    atr_14 DECIMAL(12, 2),
    bollinger_upper DECIMAL(12, 2),
    bollinger_middle DECIMAL(12, 2),
    bollinger_lower DECIMAL(12, 2),

    -- Volume
    volume_sma_20 BIGINT,
    obv BIGINT,

    PRIMARY KEY (ticker, date)
);

-- scores table (ML predictions)
CREATE TABLE scores (
    id SERIAL PRIMARY KEY,
    ticker VARCHAR(10) NOT NULL,
    date DATE NOT NULL,

    score_fundamental DECIMAL(5, 2),  -- 0-100
    score_technical DECIMAL(5, 2),
    score_consensus DECIMAL(5, 2),
    score_macro DECIMAL(5, 2),
    score_flow DECIMAL(5, 2),
    score_risk DECIMAL(5, 2),

    score_overall DECIMAL(5, 2),

    recommendation VARCHAR(20),  -- STRONG BUY, BUY, HOLD, SELL
    confidence VARCHAR(20),  -- Low, Medium, High

    -- ML specific
    probability_up_30d DECIMAL(5, 4),  -- 0-1
    probability_up_90d DECIMAL(5, 4),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(ticker, date)
);

-- Indexes
CREATE INDEX idx_prices_ticker ON prices(ticker);
CREATE INDEX idx_fundamentals_ticker ON fundamentals(ticker);
CREATE INDEX idx_consensus_ticker ON consensus(ticker);
CREATE INDEX idx_insider_ticker ON insider_trading(ticker);
CREATE INDEX idx_events_ticker ON events(ticker);
CREATE INDEX idx_technical_ticker ON technical_indicators(ticker);
CREATE INDEX idx_scores_ticker_date ON scores(ticker, date);
```

### 1.3. API Endpoints (Backend)

```python
# backend/app.py (FastAPI example)

from fastapi import FastAPI, HTTPException, Query
from fastapi.middleware.cors import CORSMiddleware
from typing import List, Optional
from datetime import date, datetime
import pandas as pd

app = FastAPI(title="Silver Bullets API", version="1.0.0")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ===== ENDPOINTS =====

@app.get("/")
def root():
    return {"message": "Silver Bullets API v1.0"}

# ----- Companies -----

@app.get("/api/companies")
def get_companies(
    pod: Optional[str] = None,
    sector: Optional[str] = None,
    min_score: Optional[float] = None
):
    """
    Retorna lista de empresas com filtros opcionais
    """
    # Query database
    # Apply filters
    # Return JSON
    pass

@app.get("/api/companies/{ticker}")
def get_company_details(ticker: str):
    """
    Retorna dados completos de uma empresa
    Inclui: fundamentals, technicals, consensus, scores
    """
    pass

# ----- Prices -----

@app.get("/api/prices/{ticker}")
def get_price_history(
    ticker: str,
    start_date: Optional[date] = None,
    end_date: Optional[date] = None,
    interval: str = "daily"  # daily, weekly, monthly
):
    """
    Retorna histórico de preços
    """
    pass

@app.get("/api/prices/{ticker}/latest")
def get_latest_price(ticker: str):
    """
    Retorna último preço conhecido
    """
    pass

# ----- Fundamentals -----

@app.get("/api/fundamentals/{ticker}")
def get_fundamentals(ticker: str):
    """
    Retorna indicadores fundamentalistas mais recentes
    """
    pass

@app.get("/api/fundamentals/{ticker}/history")
def get_fundamentals_history(
    ticker: str,
    start_date: Optional[date] = None,
    num_quarters: int = 8
):
    """
    Retorna evolução histórica de fundamentos
    """
    pass

@app.get("/api/fundamentals/sector/{sector}")
def get_sector_fundamentals(sector: str):
    """
    Retorna médias setoriais de indicadores
    """
    pass

# ----- Technical Indicators -----

@app.get("/api/technical/{ticker}")
def get_technical_indicators(ticker: str):
    """
    Retorna todos os indicadores técnicos atuais
    """
    pass

@app.get("/api/technical/{ticker}/signals")
def get_technical_signals(ticker: str):
    """
    Retorna sinais técnicos interpretados
    Ex: "RSI oversold", "MACD bullish crossover"
    """
    pass

# ----- Consensus -----

@app.get("/api/consensus/{ticker}")
def get_consensus(ticker: str):
    """
    Retorna consenso de analistas (price targets, ratings)
    Inclui: mediana, média, dispersão, revisões
    """
    pass

@app.get("/api/consensus/{ticker}/details")
def get_consensus_details(ticker: str):
    """
    Retorna lista completa de targets por casa/analista
    """
    pass

@app.get("/api/consensus/revisions")
def get_recent_revisions(
    days: int = 30,
    min_revision_pct: float = 5.0
):
    """
    Retorna targets que foram revisados recentemente
    """
    pass

# ----- Macro -----

@app.get("/api/macro/{indicator}")
def get_macro_indicator(
    indicator: str,  # SELIC, IPCA, USD_BRL, etc.
    start_date: Optional[date] = None,
    end_date: Optional[date] = None
):
    """
    Retorna série histórica de indicador macro
    """
    pass

@app.get("/api/macro/dashboard")
def get_macro_dashboard():
    """
    Retorna snapshot de todos os principais indicadores macro
    """
    pass

# ----- Events -----

@app.get("/api/events/{ticker}")
def get_company_events(
    ticker: str,
    event_type: Optional[str] = None,
    start_date: Optional[date] = None
):
    """
    Retorna eventos da empresa (earnings, dividends, news)
    """
    pass

@app.get("/api/events/upcoming")
def get_upcoming_events(days_ahead: int = 30):
    """
    Retorna próximos eventos (earnings calendar)
    """
    pass

# ----- Insider Trading -----

@app.get("/api/insider/{ticker}")
def get_insider_trading(
    ticker: str,
    months: int = 6
):
    """
    Retorna transações de insiders
    """
    pass

@app.get("/api/insider/summary")
def get_insider_summary():
    """
    Retorna resumo de insider trading (mais compras/vendas)
    """
    pass

# ----- Scores & Recommendations -----

@app.get("/api/scores/{ticker}")
def get_stock_score(ticker: str):
    """
    Retorna score multifatorial atual
    """
    pass

@app.get("/api/scores/rankings")
def get_rankings(
    score_type: str = "overall",  # overall, fundamental, technical, etc.
    pod: Optional[str] = None,
    limit: int = 20
):
    """
    Retorna ranking de ações por score
    """
    pass

@app.get("/api/recommendations")
def get_recommendations(
    recommendation: Optional[str] = None,  # STRONG BUY, BUY, etc.
    min_score: float = 70.0,
    pod: Optional[str] = None
):
    """
    Retorna ações com recomendação específica
    """
    pass

# ----- Portfolio Simulator -----

@app.post("/api/simulator/allocate")
def calculate_allocation(
    tickers: List[str],
    monthly_investment: float,
    strategy: str = "aggressive"
):
    """
    Calcula alocação ótima dado um conjunto de ações
    """
    pass

@app.post("/api/simulator/backtest")
def backtest_portfolio(
    tickers: List[str],
    weights: List[float],
    start_date: date,
    end_date: date,
    initial_capital: float = 10000.0
):
    """
    Faz backtesting de um portfólio
    Retorna: returns, sharpe, max_drawdown, etc.
    """
    pass

# ----- Screening -----

@app.post("/api/screen")
def screen_stocks(
    filters: dict
):
    """
    Screener avançado
    Ex: { "p_l__lt": 15, "roe__gt": 20, "dividend_yield__gt": 5 }
    """
    pass

# ----- Data Update Triggers (Admin) -----

@app.post("/api/admin/update/prices")
def trigger_price_update():
    """
    Dispara atualização de preços (todas as ações)
    """
    pass

@app.post("/api/admin/update/fundamentals")
def trigger_fundamentals_update():
    """
    Dispara scraping de fundamentos
    """
    pass

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## FASE 2: COLETA DE DADOS (Semanas 2-3)

### 2.1. Yahoo Finance Integration

```python
# backend/services/data_fetcher.py

import yfinance as yf
import pandas as pd
from datetime import datetime, timedelta
from typing import List, Dict
import logging

logger = logging.getLogger(__name__)

class YahooFinanceFetcher:
    """
    Fetches stock data from Yahoo Finance
    """

    def __init__(self):
        self.suffix = ".SA"  # B3 suffix

    def get_historical_prices(
        self,
        ticker: str,
        start_date: str = None,
        end_date: str = None,
        period: str = "1y"
    ) -> pd.DataFrame:
        """
        Get historical price data

        Args:
            ticker: Stock ticker (without .SA)
            start_date: Start date (YYYY-MM-DD)
            end_date: End date (YYYY-MM-DD)
            period: Period (1d, 5d, 1mo, 3mo, 6mo, 1y, 2y, 5y, 10y, ytd, max)

        Returns:
            DataFrame with columns: Date, Open, High, Low, Close, Volume, Adj Close
        """
        try:
            full_ticker = ticker + self.suffix
            stock = yf.Ticker(full_ticker)

            if start_date and end_date:
                df = stock.history(start=start_date, end=end_date)
            else:
                df = stock.history(period=period)

            if df.empty:
                logger.warning(f"No data found for {ticker}")
                return pd.DataFrame()

            # Reset index to have Date as column
            df = df.reset_index()

            # Rename columns to match our database
            df = df.rename(columns={
                'Date': 'date',
                'Open': 'open',
                'High': 'high',
                'Low': 'low',
                'Close': 'close',
                'Volume': 'volume',
                'Dividends': 'dividends',
                'Stock Splits': 'stock_splits'
            })

            df['ticker'] = ticker
            df['adjusted_close'] = df['close']  # Ya já ajusta

            return df[['ticker', 'date', 'open', 'high', 'low', 'close', 'volume', 'adjusted_close']]

        except Exception as e:
            logger.error(f"Error fetching data for {ticker}: {e}")
            return pd.DataFrame()

    def get_current_price(self, ticker: str) -> Dict:
        """
        Get current/latest price and basic info
        """
        try:
            full_ticker = ticker + self.suffix
            stock = yf.Ticker(full_ticker)
            info = stock.info

            return {
                'ticker': ticker,
                'current_price': info.get('currentPrice') or info.get('regularMarketPrice'),
                'previous_close': info.get('previousClose'),
                'open': info.get('open'),
                'day_high': info.get('dayHigh'),
                'day_low': info.get('dayLow'),
                'volume': info.get('volume'),
                'market_cap': info.get('marketCap'),
                'avg_volume': info.get('averageVolume'),
                '52w_high': info.get('fiftyTwoWeekHigh'),
                '52w_low': info.get('fiftyTwoWeekLow'),
                'timestamp': datetime.now()
            }
        except Exception as e:
            logger.error(f"Error fetching current price for {ticker}: {e}")
            return {}

    def get_fundamental_data(self, ticker: str) -> Dict:
        """
        Get fundamental data from Yahoo Finance
        Note: Yahoo data is limited for Brazilian stocks
        """
        try:
            full_ticker = ticker + self.suffix
            stock = yf.Ticker(full_ticker)
            info = stock.info

            return {
                'ticker': ticker,
                'market_cap': info.get('marketCap'),
                'enterprise_value': info.get('enterpriseValue'),
                'trailing_pe': info.get('trailingPE'),
                'forward_pe': info.get('forwardPE'),
                'peg_ratio': info.get('pegRatio'),
                'price_to_book': info.get('priceToBook'),
                'price_to_sales': info.get('priceToSalesTrailing12Months'),
                'dividend_yield': info.get('dividendYield') * 100 if info.get('dividendYield') else None,
                'payout_ratio': info.get('payoutRatio'),
                'profit_margin': info.get('profitMargins'),
                'operating_margin': info.get('operatingMargins'),
                'roe': info.get('returnOnEquity') * 100 if info.get('returnOnEquity') else None,
                'roa': info.get('returnOnAssets') * 100 if info.get('returnOnAssets') else None,
                'debt_to_equity': info.get('debtToEquity'),
                'current_ratio': info.get('currentRatio'),
                'quick_ratio': info.get('quickRatio'),
                'revenue_growth': info.get('revenueGrowth'),
                'earnings_growth': info.get('earningsGrowth'),
                'timestamp': datetime.now()
            }
        except Exception as e:
            logger.error(f"Error fetching fundamentals for {ticker}: {e}")
            return {}

    def get_dividends(self, ticker: str, period: str = "1y") -> pd.DataFrame:
        """
        Get dividend history
        """
        try:
            full_ticker = ticker + self.suffix
            stock = yf.Ticker(full_ticker)
            dividends = stock.dividends

            if dividends.empty:
                return pd.DataFrame()

            df = dividends.reset_index()
            df = df.rename(columns={'Date': 'date', 'Dividends': 'value'})
            df['ticker'] = ticker

            # Filter by period
            if period != "max":
                cutoff_date = datetime.now() - self._period_to_timedelta(period)
                df = df[df['date'] >= cutoff_date]

            return df[['ticker', 'date', 'value']]

        except Exception as e:
            logger.error(f"Error fetching dividends for {ticker}: {e}")
            return pd.DataFrame()

    def batch_fetch_prices(self, tickers: List[str]) -> Dict[str, pd.DataFrame]:
        """
        Fetch prices for multiple tickers at once
        """
        results = {}
        for ticker in tickers:
            df = self.get_historical_prices(ticker)
            if not df.empty:
                results[ticker] = df
        return results

    @staticmethod
    def _period_to_timedelta(period: str) -> timedelta:
        """Convert period string to timedelta"""
        mapping = {
            '1d': timedelta(days=1),
            '5d': timedelta(days=5),
            '1mo': timedelta(days=30),
            '3mo': timedelta(days=90),
            '6mo': timedelta(days=180),
            '1y': timedelta(days=365),
            '2y': timedelta(days=730),
            '5y': timedelta(days=1825),
            '10y': timedelta(days=3650),
        }
        return mapping.get(period, timedelta(days=365))


# Usage example
if __name__ == "__main__":
    fetcher = YahooFinanceFetcher()

    # Get historical data
    df_prices = fetcher.get_historical_prices("SUZB3", period="6mo")
    print(df_prices.head())

    # Get current price
    current = fetcher.get_current_price("SUZB3")
    print(current)

    # Get fundamentals
    fundamentals = fetcher.get_fundamental_data("SUZB3")
    print(fundamentals)
```

### 2.2. Status Invest Scraper (Fundamentals)

```python
# backend/scrapers/status_invest.py

import requests
from bs4 import BeautifulSoup
import pandas as pd
import logging
from typing import Dict, Optional

logger = logging.getLogger(__name__)

class StatusInvestScraper:
    """
    Scraper para dados do Status Invest
    IMPORTANTE: Respeitar robots.txt e rate limits!
    """

    BASE_URL = "https://statusinvest.com.br"

    def __init__(self):
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })

    def get_fundamentals(self, ticker: str) -> Dict:
        """
        Extrai indicadores fundamentalistas
        """
        try:
            url = f"{self.BASE_URL}/acoes/{ticker.lower()}"
            response = self.session.get(url, timeout=10)

            if response.status_code != 200:
                logger.warning(f"Failed to fetch {ticker}: {response.status_code}")
                return {}

            soup = BeautifulSoup(response.content, 'html.parser')

            # Exemplo de extração (HTML pode mudar!)
            data = {
                'ticker': ticker.upper(),
                'company_name': self._extract_company_name(soup),
                'sector': self._extract_sector(soup),

                # Valuation
                'p_l': self._extract_indicator(soup, 'P/L'),
                'p_vp': self._extract_indicator(soup, 'P/VP'),
                'ev_ebitda': self._extract_indicator(soup, 'EV/EBITDA'),
                'psr': self._extract_indicator(soup, 'PSR'),
                'dividend_yield': self._extract_indicator(soup, 'DY'),

                # Rentabilidade
                'roe': self._extract_indicator(soup, 'ROE'),
                'roic': self._extract_indicator(soup, 'ROIC'),
                'roa': self._extract_indicator(soup, 'ROA'),
                'margin_bruta': self._extract_indicator(soup, 'M. Bruta'),
                'margin_ebitda': self._extract_indicator(soup, 'M. EBITDA'),
                'margin_liquida': self._extract_indicator(soup, 'M. Líquida'),

                # Endividamento
                'divida_liquida_ebitda': self._extract_indicator(soup, 'Dív. Líq./EBITDA'),
                'divida_liquida_pl': self._extract_indicator(soup, 'Dív. Líq./PL'),
                'liquidez_corrente': self._extract_indicator(soup, 'Liq. Corrente'),

                # Crescimento
                'cagr_receita_5y': self._extract_indicator(soup, 'CAGR Receitas 5 anos'),
                'cagr_lucro_5y': self._extract_indicator(soup, 'CAGR Lucros 5 anos'),
            }

            return {k: v for k, v in data.items() if v is not None}

        except Exception as e:
            logger.error(f"Error scraping {ticker}: {e}")
            return {}

    def _extract_company_name(self, soup: BeautifulSoup) -> Optional[str]:
        """Extrai nome da empresa"""
        try:
            elem = soup.find('h4', class_='lh-4')
            return elem.text.strip() if elem else None
        except:
            return None

    def _extract_sector(self, soup: BeautifulSoup) -> Optional[str]:
        """Extrai setor"""
        try:
            elem = soup.find('span', text='Setor')
            if elem:
                parent = elem.find_parent()
                value = parent.find('strong')
                return value.text.strip() if value else None
        except:
            return None

    def _extract_indicator(self, soup: BeautifulSoup, label: str) -> Optional[float]:
        """
        Extrai um indicador específico
        NOTA: Este método precisa ser adaptado ao HTML real do Status Invest
        """
        try:
            # Procura pelo label
            elem = soup.find('span', text=label)
            if not elem:
                return None

            # Pega o valor (pode estar em diferentes estruturas)
            parent = elem.find_parent()
            value_elem = parent.find('strong')

            if not value_elem:
                return None

            value_text = value_elem.text.strip()

            # Remove % e R$, converte para float
            value_text = value_text.replace('%', '').replace('R$', '').replace('.', '').replace(',', '.')

            return float(value_text)

        except:
            return None

    def get_price_history_api(self, ticker: str, category: str = "acoes") -> pd.DataFrame:
        """
        Usa a API interna do Status Invest para histórico de preços
        """
        try:
            url = f"{self.BASE_URL}/{category}/tickerprice"

            payload = {
                'ticker': ticker.upper(),
                'type': 4,  # 4 = histórico completo
            }

            response = self.session.get(url, params=payload, timeout=10)

            if response.status_code != 200:
                return pd.DataFrame()

            data = response.json()

            if not data:
                return pd.DataFrame()

            df = pd.DataFrame(data)
            df['ticker'] = ticker.upper()
            df['date'] = pd.to_datetime(df['date'])

            return df

        except Exception as e:
            logger.error(f"Error fetching price history for {ticker}: {e}")
            return pd.DataFrame()


# Usage
if __name__ == "__main__":
    scraper = StatusInvestScraper()

    fundamentals = scraper.get_fundamentals("SUZB3")
    print(fundamentals)
```

### 2.3. Banco Central API (Macro Data)

```python
# backend/services/macro_data.py

import requests
import pandas as pd
from datetime import datetime, timedelta
from typing import Dict, List
import logging

logger = logging.getLogger(__name__)

class BancoCentralAPI:
    """
    API oficial do Banco Central do Brasil
    Docs: https://olinda.bcb.gov.br/olinda/servico/PTAX/versao/v1/aplicacao#!/recursos
    """

    BASE_URL = "https://api.bcb.gov.br/dados/serie/bcdata.sgs.{series_id}/dados"

    # Séries principais
    SERIES = {
        'SELIC': 432,               # Taxa Selic anualizada
        'IPCA': 433,                # IPCA mensal
        'IPCA_12M': 13522,          # IPCA acumulado 12 meses
        'IGP_M': 189,               # IGP-M
        'USD_BRL': 1,               # Dólar americano (compra)
        'USD_BRL_SELL': 10813,      # Dólar americano (venda)
        'PIB': 4385,                # PIB mensal
        'DESEMPREGO': 24369,        # Taxa de desemprego
        'PRODUCAO_INDUSTRIAL': 21859, # Produção industrial
    }

    def get_series(
        self,
        series_name: str,
        start_date: str = None,
        end_date: str = None
    ) -> pd.DataFrame:
        """
        Busca uma série temporal do BC

        Args:
            series_name: Nome da série (ex: 'SELIC', 'IPCA')
            start_date: Data inicial (DD/MM/YYYY)
            end_date: Data final (DD/MM/YYYY)
        """
        try:
            series_id = self.SERIES.get(series_name)
            if not series_id:
                logger.error(f"Unknown series: {series_name}")
                return pd.DataFrame()

            url = self.BASE_URL.format(series_id=series_id)

            params = {}
            if start_date:
                params['dataInicial'] = start_date
            if end_date:
                params['dataFinal'] = end_date

            response = requests.get(url, params=params, timeout=15)

            if response.status_code != 200:
                logger.error(f"API error: {response.status_code}")
                return pd.DataFrame()

            data = response.json()

            if not data:
                return pd.DataFrame()

            df = pd.DataFrame(data)
            df['data'] = pd.to_datetime(df['data'], format='%d/%m/%Y')
            df['valor'] = pd.to_numeric(df['valor'])
            df = df.rename(columns={'data': 'date', 'valor': 'value'})
            df['indicator'] = series_name

            return df[['indicator', 'date', 'value']]

        except Exception as e:
            logger.error(f"Error fetching {series_name}: {e}")
            return pd.DataFrame()

    def get_latest_value(self, series_name: str) -> Dict:
        """
        Retorna o valor mais recente de uma série
        """
        try:
            # Busca últimos 30 dias
            end_date = datetime.now().strftime('%d/%m/%Y')
            start_date = (datetime.now() - timedelta(days=30)).strftime('%d/%m/%Y')

            df = self.get_series(series_name, start_date, end_date)

            if df.empty:
                return {}

            latest = df.iloc[-1]

            return {
                'indicator': series_name,
                'value': latest['value'],
                'date': latest['date'],
                'change_vs_previous': None  # TODO: calcular variação
            }

        except Exception as e:
            logger.error(f"Error getting latest {series_name}: {e}")
            return {}

    def get_macro_dashboard(self) -> Dict:
        """
        Retorna snapshot de todos os principais indicadores
        """
        dashboard = {}

        for series_name in self.SERIES.keys():
            latest = self.get_latest_value(series_name)
            if latest:
                dashboard[series_name] = latest

        return dashboard


# Usage
if __name__ == "__main__":
    bc = BancoCentralAPI()

    # Get Selic history
    df_selic = bc.get_series('SELIC', start_date='01/01/2024')
    print(df_selic.tail())

    # Get latest macro data
    dashboard = bc.get_macro_dashboard()
    print(dashboard)
```

---

## FASE 3: INDICADORES TÉCNICOS (Semana 3-4)

### 3.1. Technical Indicators Calculator

```python
# backend/services/indicators.py

import pandas as pd
import pandas_ta as ta
import numpy as np
from typing import Dict, Tuple
import logging

logger = logging.getLogger(__name__)

class TechnicalIndicators:
    """
    Calcula todos os indicadores técnicos
    """

    @staticmethod
    def calculate_all(df: pd.DataFrame) -> pd.DataFrame:
        """
        Calcula todos os indicadores técnicos em um DataFrame de preços

        Args:
            df: DataFrame com colunas: date, open, high, low, close, volume

        Returns:
            DataFrame com indicadores adicionados
        """
        if df.empty or len(df) < 200:
            logger.warning("Insufficient data for technical indicators")
            return df

        df = df.copy()
        df = df.sort_values('date')

        # ----- TREND INDICATORS -----
        # Simple Moving Averages
        df['sma_20'] = ta.sma(df['close'], length=20)
        df['sma_50'] = ta.sma(df['close'], length=50)
        df['sma_200'] = ta.sma(df['close'], length=200)

        # Exponential Moving Averages
        df['ema_12'] = ta.ema(df['close'], length=12)
        df['ema_26'] = ta.ema(df['close'], length=26)

        # ----- MOMENTUM INDICATORS -----
        # RSI
        df['rsi_14'] = ta.rsi(df['close'], length=14)

        # MACD
        macd = ta.macd(df['close'], fast=12, slow=26, signal=9)
        df['macd'] = macd['MACD_12_26_9']
        df['macd_signal'] = macd['MACDs_12_26_9']
        df['macd_histogram'] = macd['MACDh_12_26_9']

        # Stochastic
        stoch = ta.stoch(df['high'], df['low'], df['close'], k=14, d=3)
        df['stochastic_k'] = stoch['STOCHk_14_3_3']
        df['stochastic_d'] = stoch['STOCHd_14_3_3']

        # Williams %R
        df['williams_r'] = ta.willr(df['high'], df['low'], df['close'], length=14)

        # ----- VOLATILITY INDICATORS -----
        # ATR
        df['atr_14'] = ta.atr(df['high'], df['low'], df['close'], length=14)

        # Bollinger Bands
        bbands = ta.bbands(df['close'], length=20, std=2)
        df['bollinger_upper'] = bbands['BBU_20_2.0']
        df['bollinger_middle'] = bbands['BBM_20_2.0']
        df['bollinger_lower'] = bbands['BBL_20_2.0']
        df['bollinger_width'] = (bbands['BBU_20_2.0'] - bbands['BBL_20_2.0']) / bbands['BBM_20_2.0'] * 100

        # ----- VOLUME INDICATORS -----
        # Volume SMA
        df['volume_sma_20'] = ta.sma(df['volume'], length=20)

        # OBV
        df['obv'] = ta.obv(df['close'], df['volume'])

        # ----- DERIVED METRICS -----
        # Price vs MAs (%)
        df['price_vs_sma50'] = (df['close'] - df['sma_50']) / df['sma_50'] * 100
        df['price_vs_sma200'] = (df['close'] - df['sma_200']) / df['sma_200'] * 100

        # Volume ratio
        df['volume_ratio'] = df['volume'] / df['volume_sma_20']

        return df

    @staticmethod
    def calculate_support_resistance(df: pd.DataFrame, window: int = 20) -> Dict:
        """
        Identifica níveis de suporte e resistência
        Usa método de pivô (local minima/maxima)
        """
        if len(df) < window * 2:
            return {'supports': [], 'resistances': []}

        df = df.copy().sort_values('date')

        # Find local minima (supports)
        df['min'] = df['low'].rolling(window=window, center=True).min()
        df['max'] = df['high'].rolling(window=window, center=True).max()

        supports = df[df['low'] == df['min']]['low'].unique()
        resistances = df[df['high'] == df['max']]['high'].unique()

        # Sort and take top 5
        supports = sorted(supports)[-5:]
        resistances = sorted(resistances, reverse=True)[:5]

        return {
            'supports': supports.tolist(),
            'resistances': resistances.tolist()
        }

    @staticmethod
    def get_technical_signals(df: pd.DataFrame) -> Dict:
        """
        Interpreta indicadores técnicos e retorna sinais
        """
        if df.empty:
            return {}

        latest = df.iloc[-1]

        signals = {
            'timestamp': latest['date'] if 'date' in latest else None,
            'price': latest['close'],

            # Trend Signals
            'trend': {
                'above_sma50': latest['close'] > latest['sma_50'] if pd.notna(latest['sma_50']) else None,
                'above_sma200': latest['close'] > latest['sma_200'] if pd.notna(latest['sma_200']) else None,
                'sma50_vs_sma200': 'Bullish' if latest['sma_50'] > latest['sma_200'] else 'Bearish',
                'trend_strength': TechnicalIndicators._calculate_trend_strength(df)
            },

            # Momentum Signals
            'momentum': {
                'rsi': latest['rsi_14'],
                'rsi_signal': TechnicalIndicators._interpret_rsi(latest['rsi_14']),
                'macd_signal': 'Bullish' if latest['macd'] > latest['macd_signal'] else 'Bearish',
                'stochastic_signal': TechnicalIndicators._interpret_stochastic(latest['stochastic_k'], latest['stochastic_d'])
            },

            # Volatility
            'volatility': {
                'atr': latest['atr_14'],
                'bollinger_position': TechnicalIndicators._bollinger_position(latest),
                'is_volatile': latest['bollinger_width'] > 10  # %
            },

            # Volume
            'volume': {
                'current': latest['volume'],
                'vs_average': latest['volume_ratio'],
                'signal': 'High Volume' if latest['volume_ratio'] > 1.5 else 'Normal'
            },

            # Overall Signal
            'overall_signal': TechnicalIndicators._calculate_overall_signal(latest)
        }

        return signals

    @staticmethod
    def _interpret_rsi(rsi: float) -> str:
        """Interpreta RSI"""
        if pd.isna(rsi):
            return "N/A"
        if rsi > 70:
            return "Overbought"
        elif rsi < 30:
            return "Oversold"
        else:
            return "Neutral"

    @staticmethod
    def _interpret_stochastic(k: float, d: float) -> str:
        """Interpreta Estocástico"""
        if pd.isna(k) or pd.isna(d):
            return "N/A"
        if k > 80:
            return "Overbought"
        elif k < 20:
            return "Oversold"
        elif k > d:
            return "Bullish Crossover"
        elif k < d:
            return "Bearish Crossover"
        else:
            return "Neutral"

    @staticmethod
    def _bollinger_position(row) -> str:
        """Posição dentro das Bandas de Bollinger"""
        price = row['close']
        upper = row['bollinger_upper']
        lower = row['bollinger_lower']
        middle = row['bollinger_middle']

        if pd.isna(upper):
            return "N/A"

        if price >= upper:
            return "Above Upper Band"
        elif price <= lower:
            return "Below Lower Band"
        elif price > middle:
            return "Above Middle"
        else:
            return "Below Middle"

    @staticmethod
    def _calculate_trend_strength(df: pd.DataFrame) -> str:
        """
        Calcula força da tendência baseado em ADX (se disponível)
        ou aproximação com MAs
        """
        # Simplified: check if MAs are aligned
        latest = df.iloc[-1]

        if pd.isna(latest['sma_50']) or pd.isna(latest['sma_200']):
            return "Insufficient Data"

        # Bullish if close > SMA50 > SMA200
        if latest['close'] > latest['sma_50'] > latest['sma_200']:
            return "Strong Uptrend"
        elif latest['close'] > latest['sma_50']:
            return "Uptrend"
        elif latest['close'] < latest['sma_50'] < latest['sma_200']:
            return "Strong Downtrend"
        elif latest['close'] < latest['sma_50']:
            return "Downtrend"
        else:
            return "Sideways"

    @staticmethod
    def _calculate_overall_signal(row) -> str:
        """
        Calcula sinal técnico geral
        """
        score = 0

        # Trend (peso 3)
        if row['close'] > row['sma_50']:
            score += 3
        if row['close'] > row['sma_200']:
            score += 2

        # Momentum (peso 2)
        if 30 < row['rsi_14'] < 70:
            score += 2  # Zona saudável
        elif row['rsi_14'] < 30:
            score += 1  # Oversold pode ser oportunidade

        if row['macd'] > row['macd_signal']:
            score += 2

        # Volume (peso 1)
        if row['volume_ratio'] > 1.2:
            score += 1

        # Classificação
        if score >= 7:
            return "STRONG BUY"
        elif score >= 5:
            return "BUY"
        elif score >= 3:
            return "HOLD"
        elif score >= 1:
            return "SELL"
        else:
            return "STRONG SELL"


# Usage
if __name__ == "__main__":
    # Load sample data
    from data_fetcher import YahooFinanceFetcher

    fetcher = YahooFinanceFetcher()
    df = fetcher.get_historical_prices("SUZB3", period="1y")

    # Calculate indicators
    df_with_indicators = TechnicalIndicators.calculate_all(df)
    print(df_with_indicators.tail())

    # Get signals
    signals = TechnicalIndicators.get_technical_signals(df_with_indicators)
    print(signals)

    # Support/Resistance
    levels = TechnicalIndicators.calculate_support_resistance(df_with_indicators)
    print(levels)
```

Vou continuar com as próximas fases...

---

Esse plano técnico está ficando muito extenso. Deixe-me criar um arquivo complementar focado em Machine Learning e Scoring.
