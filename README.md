import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import requests

# Конфигурация API (ЗАМЕНИТЕ НА РЕАЛЬНЫЕ ЗНАЧЕНИЯ)
API_ENDPOINT = "https://api.example.com/historical_matches"
API_KEY = "ВАШ_КЛЮЧ_API"

def fetch_data(team, season):
    """Получает исторические данные матчей из API."""
    try:
        params = {"team": team, "season": season}
        headers = {"X-API-Key": API_KEY}
        
        response = requests.get(API_ENDPOINT, params=params, headers=headers)
        response.raise_for_status()  # Проверка HTTP-ошибок
        
        return response.json()
    
    except requests.exceptions.HTTPError as e:
        st.error(f"Ошибка API: {e.response.status_code} - {e.response.text}")
    except requests.exceptions.RequestException as e:
        st.error(f"Ошибка соединения: {str(e)}")
    return None

def process_data(data):
    """Обрабатывает сырые данные и возвращает DataFrame."""
    if not data:
        return pd.DataFrame()
    
    processed = []
    for match in data.get("matches", []):
        try:
            processed.append({
                "date": match["date"],
                "home_team": match["home_team"]["name"],
                "away_team": match["away_team"]["name"],
                "home_score": int(match["home_score"]),
                "away_score": int(match["away_score"])
            })
        except KeyError as e:
            st.warning(f"Пропущен матч из-за отсутствия данных: {e}")
    return pd.DataFrame(processed)

def analyze_data(df):
    """Анализирует данные и выводит статистику."""
    if df.empty:
        st.warning("Нет данных для анализа")
        return
    
    # Основные метрики
    col1, col2, col3 = st.columns(3)
    with col1:
        st.metric("Всего матчей", len(df))
    with col2:
        home_win_rate = (df["home_score"] > df["away_score"]).mean()
        st.metric("Процент домашних побед", f"{home_win_rate:.1%}")
    with col3:
        avg_goals = (df["home_score"] + df["away_score"]).mean()
        st.metric("Среднее голов за матч", f"{avg_goals:.1f}")

    # Расширенная аналитика
    st.subheader("Распределение результатов")
    result_dist = df.apply(
        lambda x: "Победа дома" if x["home_score"] > x["away_score"] else 
                 "Победа гостей" if x["home_score"] < x["away_score"] else 
                 "Ничья", axis=1
    ).value_counts()
    st.bar_chart(result_dist)

def visualize_data(df):
    """Создает интерактивные визуализации."""
    if df.empty:
        return
    
    # Настройка темы
    plt.style.use("ggplot")
    
    # График распределения голов
    fig, ax = plt.subplots()
    df["total_goals"].plot(kind="hist", bins=15, ax=ax)
    ax.set_title("Распределение голов за матч")
    ax.set_xlabel("Голы")
    st.pyplot(fig)

def main():
    st.set_page_config(page_title="Sports Analyzer", layout="wide")
    st.title("📊 Анализатор спортивных данных")
    
    # Боковая панель
    with st.sidebar:
        st.header("Параметры запроса")
        team = st.text_input("Название команды", "FC Example")
        season = st.selectbox("Сезон", ["2023", "2022", "2021"])
        
        if st.button("Загрузить данные", help="Нажмите для получения данных"):
            with st.spinner("Загрузка данных..."):
                raw_data = fetch_data(team, season)
                df = process_data(raw_data)
                
            if not df.empty:
                st.session_state["data"] = df
                st.success("Данные успешно загружены!")
            else:
                st.session_state["data"] = None
    
    # Основная область
    if "data" in st.session_state and st.session_state["data"] is not None:
        df = st.session_state["data"]
        
        st.subheader("Сырые данные")
        st.dataframe(df, height=250)
        
        st.subheader("Аналитика")
        analyze_data(df)
        
        st.subheader("Визуализации")
        visualize_data(df)
    else:
        st.info("Введите параметры и нажмите 'Загрузить данные'")

if __name__ == "__main__":
    main()
