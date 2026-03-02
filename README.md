import streamlit as st
import sqlite3
import pandas as pd

conn = sqlite3.connect("change_management.db", check_same_thread=False)
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS changes(
id INTEGER PRIMARY KEY AUTOINCREMENT,
title TEXT,
description TEXT,
status TEXT
)
""")
conn.commit()

st.title("🔄 IT Change Request Management")

menu = st.sidebar.selectbox("Menu", ["New Request", "Manage Requests"])

if menu == "New Request":
    title = st.text_input("Change Title")
    desc = st.text_area("Description")

    if st.button("Submit"):
        cursor.execute("INSERT INTO changes(title,description,status) VALUES(?,?,?)",
                       (title, desc, "Pending"))
        conn.commit()
        st.success("Change Request Submitted!")

elif menu == "Manage Requests":
    df = pd.read_sql_query("SELECT * FROM changes", conn)
    st.dataframe(df)

    if not df.empty:
        selected_id = st.selectbox("Select Request ID", df["id"])
        new_status = st.selectbox("Update Status", 
                                  ["Pending", "Approved", "Rejected"])

        if st.button("Update"):
            cursor.execute("UPDATE changes SET status=? WHERE id=?",
                           (new_status, selected_id))
            conn.commit()
            st.success("Status Updated Successfully!")# Streamlit-app
