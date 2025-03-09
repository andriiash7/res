import streamlit as st
import pandas as pd

# Load the database
def load_data():
    file_path = "База_даних_резидентів.xlsx"
    try:
        df = pd.read_excel(file_path)
        return df
    except FileNotFoundError:
        st.error(f"Файл {file_path} не знайдено. Переконайтесь, що він існує.")
        return pd.DataFrame()

df = load_data()

# Streamlit UI
st.title("Інтерактивна база даних резидентів")

if df.empty:
    st.warning("Дані не завантажено. Завантажте файл та спробуйте ще раз.")
else:
    # Search bar
    search_query = st.text_input("🔍 Пошук за ім'ям або ID:")

    # Filter options
    st.sidebar.header("Фільтри")
    level_filter = st.sidebar.multiselect("Рівень резидента", df["Рівень резидента / Громадянство"].dropna().unique())
    squad_filter = st.sidebar.multiselect("Загін", df["Загін"].dropna().unique())

    # Apply filters
    filtered_df = df.copy()
    if search_query:
        filtered_df = filtered_df[df["Повне_імя"].astype(str).str.contains(search_query, case=False, na=False) | df["ID"].astype(str).str.contains(search_query, na=False)]
    if level_filter:
        filtered_df = filtered_df[filtered_df["Рівень резидента / Громадянство"].isin(level_filter)]
    if squad_filter:
        filtered_df = filtered_df[filtered_df["Загін"].isin(squad_filter)]

    # Show filtered data
    if not filtered_df.empty:
        st.dataframe(filtered_df)
        
        # Show details when a resident is selected
        selected_resident = st.selectbox("Оберіть резидента для перегляду деталей:", filtered_df["Повне_імя"].unique())
        resident_details = filtered_df[filtered_df["Повне_імя"] == selected_resident]
        st.write(resident_details.T)
    else:
        st.write("Немає даних для відображення.")
