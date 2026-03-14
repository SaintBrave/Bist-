import streamlit as st
import yfinance as yf
import pandas as pd
import pandas_ta as ta
import numpy as np
import warnings

warnings.filterwarnings('ignore')

# Sayfa Ayarları (Mobil uyumlu genişlik)
st.set_page_config(page_title="BIST Stratejik Radar", layout="wide")

st.title("🏹 BIST Stratejik Analiz Paneli")
st.write("Kemik Dipler, Destekler ve Çok Zamanlı Formasyonlar")

def get_complete_bist_list():
    return [
        "THYAO", "PGSUS", "ISCTR", "EREGL", "KONTR", "SISE", "TUPRS", "KCHOL", "AKBNK", "ASELS",
        "SASA", "HEKTS", "BIMAS", "GARAN", "SAHOL", "YKBNK", "GUBRF", "ARCLK", "FROTO", "TOASO",
        "EKGYO", "KOZAL", "PETKM", "DOHOL", "TCELL", "TTKOM", "ASTOR", "ALARK", "MIATK", "ODAS"
    ]

def formasyon_tara(df, timeframe_name):
    bulunanlar = []
    if len(df) < 60: return bulunanlar
    close, high, low = df['Close'], df['High'], df['Low']
    son_f = float(close.iloc[-1])

    # 1. W Formasyonu
    l_tail = low.tail(40)
    d1, d2 = l_tail.iloc[0:15].min(), l_tail.iloc[-15:].min()
    if abs(d1 - d2) / d1 < 0.03 and son_f > d2:
        direnc = high.tail(40).max()
        hedef = direnc + (direnc - d2)
        bulunanlar.append({"Hisse": "", "Formasyon": "İkili Dip (W)", "Periyot": timeframe_name, "Güncel": son_f, "Alım": d2, "Hedef": hedef, "Potansiyel": ((hedef/son_f)-1)*100})

    # 2. TOBO
    if len(df) > 60:
        sol, bas, sag = low.iloc[-60:-40].min(), low.iloc[-40:-20].min(), low.iloc[-20:].min()
        if bas < sol and bas < sag and abs(sol - sag) / sol < 0.05:
            direnc = high.iloc[-60:].max()
            hedef = direnc + (direnc - bas)
            bulunanlar.append({"Hisse": "", "Formasyon": "TOBO", "Periyot": timeframe_name, "Güncel": son_f, "Alım": sag, "Hedef": hedef, "Potansiyel": ((hedef/son_f)-1)*100})

    return bulunanlar

# Buton ile taramayı başlat
if st.button('🚀 Analizi Başlat / Yenile'):
    hisseler = [s + ".IS" for s in get_complete_bist_list()]
    pusu_data = []
    formasyon_data = []
    
    bar = st.progress(0)
    for i, h in enumerate(hisseler):
        try:
            df = yf.download(h, period="5y", interval="1d", auto_adjust=True, progress=False)
            if df.empty: continue
            df.columns = [col[0] if isinstance(col, tuple) else col for col in df.columns]
            son_f = float(df['Close'].iloc[-1])

            # Stratejik Hesaplar
            k_h, k_l = df['High'].rolling(26).max().iloc[-1], df['Low'].rolling(26).min().iloc[-1]
            kijun = (k_h + k_l) / 2
            sma200, sma500 = ta.sma(df['Close'], 200).iloc[-1], ta.sma(df['Close'], 500).iloc[-1]
            h1y, l1y = df['High'].tail(252).max(), df['Low'].tail(252).min()
            fib618 = h1y - (h1y - l1y) * 0.618
            y_log = np.log(df['Close'].values); x = np.arange(len(y_log))
            slope, intercept = np.polyfit(x, y_log, 1); std = np.std(y_log - (slope * x + intercept))
            log_d = np.exp((slope * len(y_log) + intercept) - (2.8 * std))
            log_z = np.exp((slope * len(y_log) + intercept) + (2.0 * std))
            h_dip = min([log_d, sma500])

            pusu_data.append({
                "Hisse": h.replace(".IS",""), "Fiyat": son_f, "Pusu": round(h_dip,2), 
                "Kijun": round(kijun,2), "SMA200": round(sma200,2), "Fib618": round(fib618,2),
                "Yıllık Zirve": round(h1y,2), "Log Zirve": round(log_z,2), "Pot%": round(((log_z/son_f)-1)*100,1)
            })

            # Formasyonlar
            for f in formasyon_tara(df, "1G"):
                f["Hisse"] = h.replace(".IS","")
                formasyon_data.append(f)
                
            df_h = yf.download(h, period="1mo", interval="1h", auto_adjust=True, progress=False)
            if not df_h.empty:
                df_h.columns = [col[0] if isinstance(col, tuple) else col for col in df_h.columns]
                for f in formasyon_tara(df_h, "1S"):
                    f["Hisse"] = h.replace(".IS","")
                    formasyon_data.append(f)
        except: continue
        bar.progress((i + 1) / len(hisseler))

    # Tabloları Göster
    st.subheader("🧱 1. Bölüm: Kemik Dip ve Stratejik Seviyeler")
    st.dataframe(pd.DataFrame(pusu_data).sort_values("Pot%", ascending=False), use_container_width=True)

    st.subheader("🚀 2. Bölüm: Teknik Formasyonlar")
    if formasyon_data:
        st.table(pd.DataFrame(formasyon_data).sort_values("Potansiyel", ascending=False))
    else:
        st.write("Şu an aktif formasyon bulunamadı.")
