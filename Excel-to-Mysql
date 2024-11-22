import tkinter as tk
from tkinter import filedialog, messagebox
import pandas as pd
from sqlalchemy import create_engine, Table, Column, MetaData, String, Integer, Float, DateTime, Boolean, Text, Numeric
from sqlalchemy.exc import SQLAlchemyError
import pymysql
import re

# Fonction pour mapper les types de données français aux types SQLAlchemy avec longueur pour VARCHAR
def map_sql_type(french_type):
    type_mapping = {
        "Texte": String(255),  # Spécifier une longueur pour VARCHAR
        "Nombre entier": Integer,
        "Nombre décimal": Float,
        "Date": DateTime,
        "Coordonnées GPS": String(50),  # Stocker les coordonnées GPS en tant que chaîne
        "Booléen": Boolean
    }
    return type_mapping.get(french_type, String(255))

# Fonction pour inférer le type de données français à partir des données de la colonne
def infer_french_type(series):
    if pd.api.types.is_bool_dtype(series):
        return "Booléen"
    elif pd.api.types.is_integer_dtype(series):
        return "Nombre entier"
    elif pd.api.types.is_float_dtype(series):
        return "Nombre décimal"
    elif pd.api.types.is_datetime64_any_dtype(series):
        return "Date"
    elif is_gps_coordinate_series(series):
        return "Coordonnées GPS"
    elif pd.api.types.is_string_dtype(series):
        return "Texte"
    else:
        return "Texte"

# Fonction pour détecter si une série contient des coordonnées GPS
def is_gps_coordinate_series(series):
    gps_pattern = re.compile(r'^-?\d{1,3}\.\d+,\s*-?\d{1,3}\.\d+$')
    return series.dropna().apply(lambda x: bool(gps_pattern.match(str(x)))).all()

class ExcelToSQLApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Importer Excel vers SQL")

        self.dataframe = None
        self.file_path = ""
        self.column_types = {}
        self.new_column_names = {}

        self.create_widgets()

    def create_widgets(self):
        # Bouton pour choisir le fichier
        self.select_file_btn = tk.Button(self.root, text="Choisir un fichier Excel", command=self.select_file)
        self.select_file_btn.pack(pady=10)

        # Liste des colonnes
        self.columns_frame = tk.Frame(self.root)
        self.columns_frame.pack(pady=10)

        # Bouton pour créer la table SQL
        self.create_table_btn = tk.Button(self.root, text="Créer la table SQL", command=self.create_table)
        self.create_table_btn.pack(pady=10)
        self.create_table_btn.config(state=tk.DISABLED)

    def select_file(self):
        self.file_path = filedialog.askopenfilename(filetypes=[("Fichiers Excel", "*.xlsx;*.xls")])
        if self.file_path:
            try:
                self.dataframe = pd.read_excel(self.file_path)
                self.show_columns()
                self.create_table_btn.config(state=tk.NORMAL)
            except Exception as e:
                messagebox.showerror("Erreur", f"Impossible de lire le fichier Excel.\n{e}")

    def show_columns(self):
        # Effacer les widgets précédents
        for widget in self.columns_frame.winfo_children():
            widget.destroy()

        tk.Label(self.columns_frame, text="Attribution des types de données et renommage des colonnes :").pack()

        self.entries = []
        self.type_vars = []
        for col in self.dataframe.columns:
            frame = tk.Frame(self.columns_frame)
            frame.pack(fill=tk.X, padx=5, pady=2)

            # Label du nom actuel de la colonne
            tk.Label(frame, text=f"Colonne actuelle: {col}", width=30, anchor='w').pack(side=tk.LEFT)

            # Champ pour le nouveau nom
            new_name_var = tk.StringVar(value=col)
            tk.Entry(frame, textvariable=new_name_var).pack(side=tk.LEFT, padx=5)
            self.entries.append(new_name_var)

            # Inférer le type de données français
            inferred_type = infer_french_type(self.dataframe[col])

            # Menu déroulant pour le type de données en français
            type_var = tk.StringVar(value=inferred_type)
            option_menu = tk.OptionMenu(
                frame,
                type_var,
                "Texte",
                "Nombre entier",
                "Nombre décimal",
                "Date",
                "Coordonnées GPS",
                "Booléen"
            )
            option_menu.pack(side=tk.LEFT, padx=5)
            self.type_vars.append(type_var)

        # Redimensionner la fenêtre pour s'adapter au contenu
        self.root.update_idletasks()
        self.root.geometry(f"{self.root.winfo_width()}x{self.root.winfo_height()}")

    def create_table(self):
        # Récupérer les nouveaux noms et types
        new_names = [var.get() for var in self.entries]
        french_data_types = [var.get() for var in self.type_vars]

        # Vérifier les noms de colonnes pour éviter les conflits
        if len(set(new_names)) != len(new_names):
            messagebox.showerror("Erreur", "Les noms de colonnes doivent être uniques.")
            return

        # Vérifier que les noms de colonnes ne sont pas vides
        if any(name.strip() == "" for name in new_names):
            messagebox.showerror("Erreur", "Les noms de colonnes ne peuvent pas être vides.")
            return

        # Mettre à jour le DataFrame avec les nouveaux noms
        self.dataframe.columns = new_names

        # Demander le nom de la table
        table_name = tk.simpledialog.askstring("Nom de la table", "Entrez le nom de la table SQL à créer :")
        if not table_name or table_name.strip() == "":
            messagebox.showwarning("Attention", "Le nom de la table est requis.")
            return

        # Configurer la base de données MySQL
        database_url = "mysql+pymysql://root:@localhost:3306/sio_v1"
        engine = create_engine(database_url, echo=False)
        metadata = MetaData()

        # Définir la table
        try:
            table_columns = []
            for i in range(len(new_names)):
                column_name = new_names[i]
                data_type = french_data_types[i]

                # Gestion des types de données spécifiques
                if data_type == "Coordonnées GPS":
                    column_type = String(50)  # Stocker les coordonnées GPS en tant que chaîne
                else:
                    column_type = map_sql_type(data_type)

                # Vérifier si le nom de la colonne est un mot réservé
                if is_sql_reserved_word(column_name):
                    messagebox.showerror("Erreur", f"Le nom de colonne '{column_name}' est un mot réservé SQL.")
                    return

                table_columns.append(Column(column_name, column_type, nullable=True))

            # Vérifier si le nom de la table est un mot réservé
            if is_sql_reserved_word(table_name):
                messagebox.showerror("Erreur", f"Le nom de table '{table_name}' est un mot réservé SQL.")
                return

            table = Table(table_name, metadata, *table_columns)
            metadata.create_all(engine)
        except SQLAlchemyError as e:
            messagebox.showerror("Erreur SQLAlchemy", f"Erreur lors de la création de la table.\n{e}")
            return

        # Gérer les valeurs NaN en les remplaçant par None
        self.dataframe = self.dataframe.where(pd.notnull(self.dataframe), None)

        # Convertir les colonnes de coordonnées GPS en chaînes
        for i in range(len(french_data_types)):
            if french_data_types[i] == "Coordonnées GPS":
                self.dataframe[new_names[i]] = self.dataframe[new_names[i]].astype(str)

        # Insérer les données
        try:
            df_records = self.dataframe.to_dict(orient='records')

            # Utiliser engine.begin() pour gérer la transaction
            with engine.begin() as connection:
                connection.execute(table.insert(), df_records)
            messagebox.showinfo("Succès", f"La table '{table_name}' a été créée et remplie avec succès.")
        except Exception as e:
            messagebox.showerror("Erreur", f"Erreur lors de l'insertion des données.\n{e}")
            # Afficher l'exception complète dans la console pour le débogage
            import traceback
            traceback.print_exc()

# Fonction pour vérifier si un mot est un mot réservé SQL
def is_sql_reserved_word(word):
    sql_reserved_words = set([
        'select', 'insert', 'update', 'delete', 'from', 'where', 'join',
        'inner', 'left', 'right', 'on', 'and', 'or', 'not', 'group', 'by',
        'order', 'limit', 'table', 'create', 'drop', 'alter', 'index', 'if',
        'exists', 'null', 'like', 'into', 'values', 'set'
    ])
    return word.lower() in sql_reserved_words

if __name__ == "__main__":
    root = tk.Tk()
    app = ExcelToSQLApp(root)
    root.mainloop()
