import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import norm
import mpld3
from matplotlib.backends.backend_pdf import PdfPages
import plotly.express as px
#import ptitprince as pt
from textwrap import wrap
import plotly.graph_objects as go
import os
from mpl_toolkits.axes_grid1.inset_locator import inset_axes
import math

### CTMT ###

# Analisar dados de energia do circuito de média tensão#
csv_path_ctmt = r"C:\TCC_V02\BDGD_2023\CTMT.csv"

CTMT = pd.read_csv(csv_path_ctmt, sep=';', encoding='latin1')

ENE_colunas = [f'ENE_{str(i).zfill(2)}' for i in range(1, 13)]

dados_ene_ctmt = CTMT[ENE_colunas].copy()

for col in ENE_colunas:
    dados_ene_ctmt[col] = (dados_ene_ctmt[col].astype(str).str.replace(',', '.', regex=False))
    dados_ene_ctmt = dados_ene_ctmt.apply(pd.to_numeric, errors='coerce')

dados_sem_zero_ctmt = dados_ene_ctmt.mask(dados_ene_ctmt == 0)

ctmt_mean_mes = dados_sem_zero_ctmt.mean(skipna=True)
ctmt_max_mes = dados_sem_zero_ctmt.max(skipna=True)

valores_ctmt = dados_sem_zero_ctmt.values.flatten()
valores_ctmt = valores_ctmt[~np.isnan(valores_ctmt)]

ctmt_max_geral = np.max(valores_ctmt)
ctmt_mean_geral = np.mean(valores_ctmt)

saida_txt_ctmt = os.path.join(os.path.dirname(csv_path_ctmt), "ctmt.txt")

with open(saida_txt_ctmt, "w", encoding="utf-8") as f:

    f.write("CTMT")
    f.write(f"Valor máximo: {ctmt_max_geral:.2f} kWh")
    f.write(f"Média: {ctmt_mean_geral:.2f} kWh")

    f.write("Média mensal:")
    for col, val in ctmt_mean_mes.items():
        f.write(f"{col}: {val:.2f} kWh")


### UCBT ###

# Análise de dados de energia das unidades consumidoraas de média tensão #
csv_path = r"C:\TCC_V02\BDGD_2023\UCBT_tab.csv"

UCBT = pd.read_csv(csv_path, sep=';')

colunas_energia = [f'ENE_{str(i).zfill(2)}' for i in range(1, 13)]
dados_energia_UCBT = UCBT[colunas_energia]

dados_sem_zero = dados_energia_UCBT.replace(0, pd.NA)

ucbt_max_por_mes = dados_sem_zero.max(skipna=True)
ucbt_mean_por_mes = dados_sem_zero.mean(skipna=True)

valores_ucbt = dados_sem_zero.values.flatten()
valores_ucbt = valores_ucbt[~pd.isna(valores_ucbt)]

q1_ucbt = np.quantile(valores_ucbt, 0.25)
median_ucbt = np.median(valores_ucbt)
q3_ucbt = np.quantile(valores_ucbt, 0.75)
max_ucbt = np.max(valores_ucbt)
mean_ucbt = np.mean(valores_ucbt)

print("Resultados UCBT")
print("Média:", ucbt_mean_por_mes)
print("Valor máximo:", ucbt_max_por_mes)

saida_txt = os.path.join(os.path.dirname(csv_path), "resultados_ucbt.txt")
with open(saida_txt, "w", encoding="utf-8") as f:

    f.write("UCBT")
    f.write(f"Q1: {q1_ucbt:.2f} kWh")
    f.write(f"Q2: {median_ucbt:.2f} kWh")
    f.write(f"Q3: {q3_ucbt:.2f} kWh")
    f.write(f"Valor máximo: {max_ucbt:.2f} kWh")
    f.write(f"Média: {mean_ucbt:.2f} kWh")

    f.write("Média:")
    for col, val in ucbt_mean_por_mes.items():
        f.write(f"{col}: {val:.2f} kWh")

    f.write("Valor máximo:")
    for col, val in ucbt_max_por_mes.items():
        f.write(f"{col}: {val:.2f} kWh")

print(f"Arquivo salvo: {saida_txt}")

# Analisar indicadores de qualidade de serviço #
UCBTs = pd.read_csv(r'C:\TCC_V02\BDGD_2023\UCBT_tab.csv', sep=';')

feeders = sorted(UCBTs["CTMT"].astype(str).unique())
LOC = sorted(UCBTs["ARE_LOC"].astype(str).unique())
contagem_nu = UCBTs['ARE_LOC'].value_counts().get('NU', 0)
contagem_UB = UCBTs['ARE_LOC'].value_counts().get('UB', 0)

all_data = []

for mes in range(1, 13):
    mes_str = f'{mes:02}'
    
    for loc in LOC:
        y_data = UCBTs[UCBTs["ARE_LOC"] == loc][f"DIC_{mes_str}"].values
        month_data = pd.DataFrame({'Month': [mes_str] * len(y_data), 'Local': [loc] * len(y_data),'DIC': y_data})
        all_data.append(month_data)

data_combined = pd.concat(all_data)

plt.figure(figsize=(12.5, 3.0))

data_combined.reset_index(drop=True, inplace=True)

data_plot = data_combined[data_combined['Local'].isin(['NU', 'UB'])]

# Criar boxplot #
sns.boxplot(x='Month', y='DIC', hue='Local', data=data_plot, fliersize=2, palette={'NU':'#1f77b4', 'UB': '#aec7e8'})
plt.xticks(rotation=90)
plt.xlabel('MÊS', fontsize=11, fontname="Times New Roman")
plt.ylabel('DIC', fontsize=11, fontname="Times New Roman")
plt.tight_layout()
plt.show()

all_data = []

for mes in range(1, 13):
    mes_str = f'{mes:02}'
    
    for loc in LOC:
        y_data = UCBTs[UCBTs["ARE_LOC"] == loc][f"FIC_{mes_str}"].values
        month_data = pd.DataFrame({'Month': [mes_str] * len(y_data), 'Local': [loc] * len(y_data), 'FIC': y_data})
        all_data.append(month_data)

data_combined = pd.concat(all_data)

# Configurar boxplot #
plt.figure(figsize=(12.5, 3.0))

data_combined.reset_index(drop=True, inplace=True)

data_plot = data_combined[data_combined['Local'].isin(['NU', 'UB'])]

# Criar boxplot #
sns.boxplot(x='Month', y='FIC', hue='Local', data=data_plot, fliersize=2, palette={'NU': '#1f77b4', 'UB': '#aec7e8'})
plt.xticks(rotation=90)
plt.xlabel('MÊS', fontsize=11, fontname="Times New Roman")
plt.ylabel('FIC', fontsize=11, fontname="Times New Roman")
plt.tight_layout()
plt.show()

for mes in range(1, 13):

    if mes < 10:
        mes = f'0{mes}'

    y_data = [UCBTs[UCBTs["CTMT"] == feeder][f"DIC_{mes}"].values for feeder in feeders]
    data_combined = pd.DataFrame({'Feeder': np.repeat(feeders, [len(data) for data in y_data]), f'DIC_{mes}': np.concatenate(y_data)})

    plt.figure(figsize=(12.5, 3.0))

    # Criar boxplot #
    sns.boxplot(x='Feeder', y=f'DIC_{mes}', data=data_combined, palette='Blues',fliersize=3)
    plt.xticks(rotation=90)
    plt.xlabel('Alimentador', fontsize=5, fontname="Times New Roman")
    plt.ylabel(f'DIC_{mes}', fontsize=5, fontname="Times New Roman")
    plt.tight_layout()
    plt.savefig(f'boxplot_DIC_{mes}.png', dpi=800, bbox_inches='tight')
    plt.close()

# Extrair todos os valores de DIC #
DIC_01 = UCBTs["DIC_01"].values
DIC_02 = UCBTs["DIC_02"].values
DIC_03 = UCBTs["DIC_03"].values
DIC_04 = UCBTs["DIC_04"].values
DIC_05 = UCBTs["DIC_05"].values
DIC_06 = UCBTs["DIC_06"].values
DIC_07 = UCBTs["DIC_07"].values
DIC_08 = UCBTs["DIC_08"].values
DIC_09 = UCBTs["DIC_09"].values
DIC_10 = UCBTs["DIC_10"].values
DIC_11 = UCBTs["DIC_11"].values
DIC_12 = UCBTs["DIC_12"].values

# Listar todas as colunas de DIC #
DIC_cols = [f"DIC_{str(i).zfill(2)}" for i in range(1, 13)]

for col in DIC_cols:
    UCBTs[col] = UCBTs[col].clip(upper=10)

fig, axes = plt.subplots(4, 3, figsize=(15, 10))
axes = axes.flatten()

for i, col in enumerate(DIC_cols):
    # Remover valores nulos #
    values = UCBTs[col].dropna()
    
    # Criar gráfico #
    axes[i].hist(values, bins=10, color='#1f77b4', edgecolor='black', alpha=0.7)
    axes[i].set_title(col, fontsize=10, fontname="Times New Roman")
    axes[i].set_xlabel('Duração [horas]', fontsize=10)
    axes[i].set_ylabel('Frequência', fontsize=10)
    axes[i].tick_params(axis='both', labelsize=10)
    axes[i].set_xlim(-0.5, 10)
    
    # Ajustar o zoom #
    ax_inset = inset_axes(axes[i], width="40%", height="40%", loc="upper right", borderpad=1.5)
    
    zoom_data = values[values >= 2]
    
    # Plotar gráfico do zoom #
    ax_inset.hist(zoom_data, bins=6, color='#1f77b4', edgecolor='black', alpha=0.7)
    
    ax_inset.set_xlim(0, 10)
    ax_inset.set_xticks([0, 2, 4, 6, 8, 10])
    ax_inset.tick_params(axis='both', labelsize=6)
    ax_inset.set_yticks([])
    ax_inset.set_facecolor("#fce8dc")
    
    for spine in ax_inset.spines.values():
        spine.set_edgecolor('black')
        spine.set_linewidth(0.8)


plt.subplots_adjust(left=0.08, right=0.95, top=0.95, bottom=0.05, hspace=0.6, wspace=0.3)
plt.savefig('DIC_ano.png', dpi=800, bbox_inches='tight', pad_inches=0.05, facecolor='white')
plt.show()

meses_DIC = [f"DIC_{str(i).zfill(2)}" for i in range(1, 13)]
y_data_DIC = []

for mes in meses_DIC:
    y_data_DIC.extend(UCBTs[mes].values)

min_DIC = min(y_data_DIC)
max_DIC = max(y_data_DIC)
mean_DIC = sum(y_data_DIC)/len(y_data_DIC)

print(f"Resultados DIC")
print(f"Mínimo: {min_DIC}")
print(f"Máximo: {max_DIC}")
print(f"Média: {mean_DIC:.2f}")

# Extrair todos os valores de FIC #
FIC_01 = UCBTs["FIC_01"].values
FIC_02 = UCBTs["FIC_02"].values
FIC_03 = UCBTs["FIC_03"].values
FIC_04 = UCBTs["FIC_04"].values
FIC_05 = UCBTs["FIC_05"].values
FIC_06 = UCBTs["FIC_06"].values
FIC_07 = UCBTs["FIC_07"].values
FIC_08 = UCBTs["FIC_08"].values
FIC_09 = UCBTs["FIC_09"].values
FIC_10 = UCBTs["FIC_10"].values
FIC_11 = UCBTs["FIC_11"].values
FIC_12 = UCBTs["FIC_12"].values

# Listar todas as colunas de FIC #
FIC_cols = [f"FIC_{str(i).zfill(2)}" for i in range(1, 13)]

for col in FIC_cols:
    UCBTs[col] = UCBTs[col].clip(upper=10)

fig, axes = plt.subplots(4, 3, figsize=(15, 10))
axes = axes.flatten()

for i, col in enumerate(FIC_cols):

    values = UCBTs[col].dropna()

# Plotar grafico #
    axes[i].hist(values, bins=10, color='#1f77b4', edgecolor='black')
    axes[i].set_title(col, fontsize=10, fontname="Times New Roman")
    axes[i].set_xlabel('Quantidade de Interrupções', fontsize=10)
    axes[i].set_ylabel('Frequência', fontsize=10)
    axes[i].tick_params(axis='both', labelsize=10)
    axes[i].set_xlim(-0.5, 10)

# Criar zoom dentro do gráfico #
    ax_inset = inset_axes(axes[i], width="45%", height="45%", loc="upper right", borderpad=1)
    zoom_data = values[values >= 2]

# Plotar gráfico do zoom #
    ax_inset.hist(zoom_data, bins=6, color='#1f77b4', edgecolor='black')
    ax_inset.set_xlim(0, 10)
    ax_inset.set_xticks([0, 2, 4, 6, 8, 10])
    ax_inset.tick_params(axis='both', labelsize=7)
    ax_inset.set_yticks([])
    ax_inset.set_facecolor("#fce8dc")

# plotar e salvar gráfico #
plt.subplots_adjust(left=0.08, right=0.95, top=0.95, bottom=0.05, hspace=0.6, wspace=0.3)
plt.savefig('FIC_ano.png', dpi=800, bbox_inches='tight', pad_inches=0.05, facecolor='white')
plt.show()


meses_FIC = [f"FIC_{str(i).zfill(2)}" for i in range(1, 13)]
y_data_FIC = []

for mes in meses_FIC:
    y_data_FIC.extend(UCBTs[mes].values)

min_FIC = min(y_data_FIC)
max_FIC = max(y_data_FIC)
mean_FIC = sum(y_data_FIC)/len(y_data_FIC)
y_data_df = pd.DataFrame(y_data_FIC, columns=["Valores_FIC"])

print(f"Resultados FIC")
print(f"Mínimo: {min_FIC}")
print(f"Máximo: {max_FIC}")
print(f"Média: {mean_FIC:.2f}")

# ### UCMT ###

# csv_path_ucmt = r'C:\TCC_V02\BDGD_2023\UCMT_tab.csv'

# UCMT = pd.read_csv(csv_path_ucmt, sep=';', encoding='latin1')

# colunas_energia = [f'ENE_{str(i).zfill(2)}' for i in range(1, 13)]

# dados_energia_UCMT = UCMT[colunas_energia].copy()

# for col in colunas_energia:
#     dados_energia_UCMT[col] = (dados_energia_UCMT[col].astype(str).str.replace(',', '.', regex=False).astype(float))

# dados_energia_UCMT.replace(0, np.nan, inplace=True)

# ucmt_mean_por_mes = dados_energia_UCMT.mean()
# ucmt_max_por_mes = dados_energia_UCMT.max()

# valores_ucmt = dados_energia_UCMT.values.flatten()
# valores_ucmt = valores_ucmt[~np.isnan(valores_ucmt)]

# q1_ucmt = np.quantile(valores_ucmt, 0.25)
# median_ucmt = np.median(valores_ucmt)
# q3_ucmt = np.quantile(valores_ucmt, 0.75)
# max_ucmt = np.max(valores_ucmt)
# mean_ucmt = np.mean(valores_ucmt)

# print("Resultados UCMT")
# print("Média:", ucmt_mean_por_mes)
# print("Valor máximo:", ucmt_max_por_mes)

# saida_txt_ucmt = os.path.join(os.path.dirname(csv_path_ucmt), "resultados_ucmt.txt")

# with open(saida_txt_ucmt, "w", encoding="utf-8") as f:

#     f.write("UCMT")

#     f.write(f"Primeiro Quartil (Q1): {q1_ucmt:.2f} kWh")
#     f.write(f"Mediana (Q2): {median_ucmt:.2f} kWh")
#     f.write(f"Terceiro Quartil (Q3): {q3_ucmt:.2f} kWh")
#     f.write(f"Valor máximo: {max_ucmt:.2f} kWh")
#     f.write(f"Média: {mean_ucmt:.2f} kWh")

#     f.write("Média mensal:")
#     for col, val in ucmt_mean_por_mes.items():
#         f.write(f"{col}: {val:.2f} kWh")

#     f.write("Valor máximo:")
#     for col, val in ucmt_max_por_mes.items():
#         f.write(f"{col}: {val:.2f} kWh")

# print(f"Arquivo UCMT salvo em: {saida_txt_ucmt}")

# # Gráficos de UCMT, UGBT, Quant. de transformadores e distribuição da pot. nominal com relação aos transformadores #

# # Gráfico de pizza da potencia dos transformadores #

# base = r"C:\TCC_V02\BDGD_2023"

# transf_path = os.path.join(base, "UNTRMT.csv")

# TRANSF = pd.read_csv(transf_path, sep=";", encoding="latin1")

# TRANSF["POT_NOM"] = pd.to_numeric(TRANSF["POT_NOM"], errors="coerce")

# contagem_pot = TRANSF["POT_NOM"].value_counts().sort_index()

# LIMITE_OUTROS = 250

# pot = contagem_pot[contagem_pot < LIMITE_OUTROS].index

# TRANSF["POT_AGRUPADA"] = TRANSF["POT_NOM"].apply(lambda x: "Outros" if x in pot else int(x))

# contagem_final = (TRANSF["POT_AGRUPADA"].value_counts().reset_index())

# contagem_final.columns = ["Potência (kVA)", "Quantidade"]

# if "Outros" in contagem_final["Potência (kVA)"].values:
#     outros = contagem_final[contagem_final["Potência (kVA)"] = "Outros"]
#     contagem_final = contagem_final[contagem_final["Potência (kVA)"] != "Outros"]
#     contagem_final = pd.concat([contagem_final, outros], ignore_index=True)

# # Criar gráfico #

# fig = px.pie(contagem_final, values="Quantidade", names="Potência (kVA)", hole=0.45, color_discrete_sequence=px.colors.sequential.Blues)

# fig.update_traces(textposition="inside", textinfo="label+value+percent", textfont_size=16)

# fig.update_layout(showlegend=True)

# idx_max = contagem_final["Quantidade"].idxmax()
# pot_max = contagem_final.loc[idx_max, "Potência (kVA)"]
# qtd_max = contagem_final.loc[idx_max, "Quantidade"]

# angle = 360 * idx_max / len(contagem_final)
# angle_rad = math.radians(angle)

# fig.add_annotation(x=0.7 * math.cos(angle_rad), y=0.7 * math.sin(angle_rad), text=f"{pot_max} kVA({qtd_max})", showarrow=True, arrowhead=2, font=dict(size=14, color="black"))

# fig.write_image("transformadores_potencia.png", scale=1, engine="kaleido")
# fig.show()

# # Adicionar bloxplot quantidade de transformadores #
# TRAFOs = [135,
# 166,
# 8,
# 96,
# 53,
# 154,
# 134,
# 59,
# 2,
# 30,
# 227,
# 111,
# 3,
# 458,
# 35,
# 416,
# 112,
# 176,
# 115,
# 187,
# 46,
# 171,
# 81,
# 105,
# 185,
# 16,
# 155,
# 253,
# 233,
# 381,
# 435,
# 905,
# 80,
# 120,
# 157,
# 71,
# 138,
# 51,
# 81,
# 108,
# 42,
# 270,
# 11,
# 12,
# 18,
# 174,
# 250,
# 173,
# 322,
# 156,
# 227,
# 209,
# 185,
# 219,
# 152,
# 948,
# 383,
# 135,
# 107,
# 155,
# 65,
# 149]

# df = pd.DataFrame(TRAFOs, columns=['Valores'])

# fig, ax = plt.subplots(figsize=(5, 2))

# sns.violinplot(x=df['Valores'], scale='area', inner=None, ax=ax, width=0.5, palette='Blues')

# y = np.random.uniform(low=-0.05, high=0.05, size=len(df)) 
# ax.scatter(df['Valores'], y, color='blue', alpha=0.6)
# ax.set_xlim(0, 1200)

# # Adicionar boxplot #
# ax.boxplot([df['Valores']], vert=False, positions=[0], manage_ticks=False, showfliers=False, showcaps=False, medianprops={'color': 'red', 'linewidth': 2}, whiskerprops={'color': 'black', 'linewidth': 1.5}, boxprops={'color': 'black', 'linewidth': 1.5})

# ax.set_xlabel("Quantidade de Transformadores", fontsize=10)
# ax.set_yticks([0])  
# plt.tight_layout()

# # Calcular percentis de transformadores #
# q1 = df['Valores'].quantile(0.25)
# median = df['Valores'].median()
# q3 = df['Valores'].quantile(0.75)
# max_val = df['Valores'].max()
# mean_val = df['Valores'].mean()

# # Mostrar dados #
# print(f"Primeiro Quartil (Q1): {q1}")
# print(f"Mediana (Q2): {median}")
# print(f"Terceiro Quartil (Q3): {q3}")
# print(f"Valor máximo: {max_val}")
# print(f"Média: {mean_val}")

# # Gerar bloxplot UCBTs #
# UCs = [8613,
# 10407,
# 29,
# 615,
# 2322,
# 9029,
# 8421,
# 3646,
# 4,
# 152,
# 9553,
# 6616,
# 52,
# 806,
# 942,
# 10645,
# 753,
# 7490,
# 9233,
# 7347,
# 3016,
# 10200,
# 3897,
# 6611,
# 15807,
# 342,
# 9559,
# 13765,
# 13889,
# 4723,
# 18116,
# 17831,
# 1045,
# 4387,
# 8146,
# 3378,
# 10711,
# 3329,
# 4196,
# 7276,
# 2884,
# 15552,
# 1195,
# 1467,
# 899,
# 4815,
# 16696,
# 6189,
# 2062,
# 14717,
# 10960,
# 13012,
# 11406,
# 8115,
# 9966,
# 1677,
# 8576,
# 10341,
# 8712,
# 9207,
# 5573,
# 8919 
# ]

# df = pd.DataFrame(UCs, columns=['Valores'])

# fig, ax = plt.subplots(figsize=(5, 2))

# # Criar o gráfico de violino #
# sns.violinplot(x=df['Valores'], scale='area', inner=None, ax=ax, width=0.5, palette='Blues')

# y = np.random.uniform(low=-0.05, high=0.05, size=len(df)) 
# ax.scatter(df['Valores'], y, color='blue', alpha=0.6)
# ax.set_xlim(0, 22000)

# # Adicionar boxplot #
# ax.boxplot([df['Valores']], vert=False, positions=[0], manage_ticks=False, showfliers=False, showcaps=False, medianprops={'color': 'red', 'linewidth': 2}, whiskerprops={'color': 'black', 'linewidth': 1.5}, boxprops={'color': 'black', 'linewidth': 1.5})
# ax.set_xlabel("Quantidade de UCBTs", fontsize=10)
# ax.set_yticks([0]) 
# plt.tight_layout()

# # Calcular percentis UCs #
# q1 = df['Valores'].quantile(0.25)
# median = df['Valores'].median()
# q3 = df['Valores'].quantile(0.75)
# max_val = df['Valores'].max()
# mean_val = df['Valores'].mean()

# # Mostrar percentis #
# print(f"Primeiro Quartil (Q1): {q1}")
# print(f"Mediana (Q2): {median}")
# print(f"Terceiro Quartil (Q3): {q3}")
# print(f"Valor máximo: {max_val}")
# print(f"Média: {mean_val}")

# # Criar bloxplot UGs #
# UGs = [327,
# 464,
# 2,
# 43,
# 78,
# 324,
# 299,
# 175,
# 14,
# 462,
# 173,
# 1,
# 65,
# 25,
# 643,
# 14,
# 127,
# 84,
# 514,
# 33,
# 311,
# 148,
# 149,
# 472,
# 349,
# 417,
# 460,
# 290,
# 608,
# 280,
# 263,
# 388,
# 179,
# 93,
# 213,
# 64,
# 59,
# 125,
# 42,
# 427,
# 1,
# 4,
# 16,
# 418,
# 868,
# 482,
# 112,
# 255,
# 992,
# 589,
# 274,
# 339,
# 241,
# 157,
# 427,
# 279,
# 190,
# 212,
# 160,
# 290,
# ]

# df = pd.DataFrame(UGs, columns=['Valores'])

# fig, ax = plt.subplots(figsize=(5, 2))

# # Criar o gráfico de violino #
# sns.violinplot(x=df['Valores'], scale='area', inner=None, ax=ax, width=0.5, palette='Blues')

# y = np.random.uniform(low=-0.05, high=0.05, size=len(df))  
# ax.scatter(df['Valores'], y, color='blue', alpha=0.6)
# ax.set_xlim(0, 1000)

# # Adicionar boxplot #
# ax.boxplot([df['Valores']], vert=False, positions=[0], manage_ticks=False, showfliers=False, showcaps=False, medianprops={'color': 'red', 'linewidth': 2}, whiskerprops={'color': 'black', 'linewidth': 1.5}, boxprops={'color': 'black', 'linewidth': 1.5})
# ax.set_xlabel("Quantidade de UGs", fontsize=10)
# ax.set_yticks([0])  
# plt.tight_layout()
# plt.show()

# # Calcular percentis #
# q1 = df['Valores'].quantile(0.25)
# median = df['Valores'].median()
# q3 = df['Valores'].quantile(0.75)
# max_val = df['Valores'].max()
# mean_val = df['Valores'].mean()

# # mostrar percentir UGs  #
# print(f"Primeiro Quartil (Q1): {q1}")
# print(f"Mediana (Q2): {median}")
# print(f"Terceiro Quartil (Q3): {q3}")
# print(f"Valor máximo: {max_val}")
# print(f"Média: {mean_val}")

# # Gráfico UCMT #

# ucmt_alimentador = (UCMT.groupby('CTMT').size().reset_index(name='Qtd_UCMT'))

# df = ucmt_alimentador[['Qtd_UCMT']]

# fig, ax = plt.subplots(figsize=(5, 2))

# # Plotar grafico #
# sns.violinplot(x=df['Qtd_UCMT'], scale='area', inner=None, ax=ax, width=1, palette='Blues')

# y = np.random.uniform(low=-0.05, high=0.05, size=len(df))
# ax.scatter(df['Qtd_UCMT'], y, color='blue', alpha=0.6)

# ax.boxplot([df['Qtd_UCMT']], vert=False, positions=[0], manage_ticks=False, showfliers=False, showcaps=False, medianprops={'color': 'red', 'linewidth': 2}, whiskerprops={'color': 'black', 'linewidth': 1.5}, boxprops={'color': 'black', 'linewidth': 1.5})

# ax.set_xlabel("Quantidade de UCMTs", fontsize=11)
# ax.set_yticks([])
# ax.set_xlim(left=0)
# plt.tight_layout()
# plt.show()

# ### UGBT ###

# csv_path_ugbt = r'C:\TCC_V02\BDGD_2023\UGBT_tab.csv'

# UGBT = pd.read_csv(csv_path_ugbt, sep=';', encoding='latin1')

# colunas_energia = [f'ENE_{str(i).zfill(2)}' for i in range(1, 13)]

# dados_UGBT = UGBT[colunas_energia]

# # Ajustar formato para números #
# for col in colunas_energia:
#     dados_UGBT[col] = (dados_UGBT[col].astype(str).str.replace(',', '.', regex=False))
#     dados_UGBT = dados_UGBT.apply(pd.to_numeric, errors='coerce')

# dados_sem_zero_ugbt = dados_UGBT.mask(dados_UGBT = 0)

# ugbt_max_mensal = dados_sem_zero_ugbt.max(skipna=True)
# ugbt_mean_mensal = dados_sem_zero_ugbt.mean(skipna=True)

# valores_ugbt = dados_sem_zero_ugbt.values.flatten()
# valores_ugbt = valores_ugbt[~np.isnan(valores_ugbt)]

# q1_ugbt = np.quantile(valores_ugbt, 0.25)
# median_ugbt = np.median(valores_ugbt)
# q3_ugbt = np.quantile(valores_ugbt, 0.75)
# max_ugbt = np.max(valores_ugbt)
# mean_ugbt = np.mean(valores_ugbt)

# # Estatísticas por consumidor  #
# UGBT['media_consumidor'] = dados_sem_zero_ugbt.mean(axis=1, skipna=True)
# UGBT['max_consumidor'] = dados_sem_zero_ugbt.max(axis=1, skipna=True)

# # Salvar arquivo .txt #
# saida_txt_ugbt = os.path.join(os.path.dirname(csv_path_ugbt), "resultados_ugbt.txt")

# # Salvar no arquivo .txt #
# with open(saida_txt_ugbt, "w", encoding="utf-8") as f:

#     f.write("UGBT")

#     f.write(f"Primeiro Quartil (Q1): {q1_ugbt:.2f} kWh")
#     f.write(f"Mediana (Q2): {median_ugbt:.2f} kWh")
#     f.write(f"Terceiro Quartil (Q3): {q3_ugbt:.2f} kWh")
#     f.write(f"Valor máximo: {max_ugbt:.2f} kWh")
#     f.write(f"Média: {mean_ugbt:.2f} kWh")

#     f.write("Média mensal:")
#     for col, val in ugbt_mean_mensal.items():
#         f.write(f"{col}: {val:.2f} kWh")

#     f.write("Valor máximo mensal:")
#     for col, val in ugbt_max_mensal.items():
#         f.write(f"{col}: {val:.2f} kWh")

# print(f"Arquivo salvo em: {saida_txt_ugbt}")

# # Plotar grafico energia gerada UGMTs ao longo dos meses 1,6 e 12 #

# ## Dados #####

# data = {'ENE_01': [16240, 0, 0, 209300, 2640, 480, 4200, 0, 0, 6720, 10560, 0, 4000, 3880, 12560, 0, 3360, 1360, 24960, 0, 0, 3120, 2720, 10280, 13300, 0, 3440, 93100, 5520, 1760, 0, 0, 480, 350, 739200, 6040, 2660, 686000, 57400, 823200, 238000, 854000, 34230, 0, 761600, 308000, 284200, 294000, 428400],
#     'ENE_06': [8260, 0, 0, 89600, 16720, 320, 5600, 0, 4400, 7680, 11920, 0, 3520, 1760, 12320, 0, 2760, 320, 8400, 19950, 0, 1160, 1040, 4360, 3850, 12600, 1760, 32900, 2800, 1600, 0, 0, 0, 140, 708400, 3680, 1610, 624400, 74900, 800800, 221200, 828800, 33180, 0, 686000, 302400, 218400, 305200, 422800],
#     'ENE_12': [8050, 0, 0, 110285, 17756, 368, 4830, 0, 10764, 4692, 10672, 0, 6348, 3312, 10994, 0, 4922, 368, 17112, 13282.5, 6934.404, 3312, 4048, 22218, 7245, 20447, 3496, 36225, 6348, 1932, 12006, 8050, 92, 80.5, 785680, 5842, 2898, 721280, 88550, 914480, 236670, 756700, 26082, 0, 817880, 323610, 333270, 305900, 408940]}

# df = pd.DataFrame(data)

# df_melted = df.melt(var_name='Mês', value_name='Valores')

# fig, axes = plt.subplots(nrows=3, ncols=1, figsize=(6, 8), sharex=True)

# # Plotar os gráficos separados #
# for i, (month, ax) in enumerate(zip(df.columns, axes.flatten())):
#     sns.violinplot(x='Valores', data=df_melted[df_melted['Mês'] == month], density_norm='area', inner=None, ax=ax, palette='Blues', legend=False)
    
#     y = np.random.uniform(low=-0.2, high=0.2, size=len(df_melted[df_melted['Mês'] == month])) 
#     ax.scatter(df_melted[df_melted['Mês'] == month]['Valores'], y, color='blue', alpha=0.6)
    
#     # Adicionar boxplot #
#     ax.boxplot([df[month]], vert=False, positions=[0], manage_ticks=False, showfliers=False, showcaps=False, medianprops={'color': 'red', 'linewidth': 2}, whiskerprops={'color': 'black', 'linewidth': 1.5}, boxprops={'color': 'black', 'linewidth': 1.5})
#     ax.set_xlim(0, 1.2e6)
#     ax.set_ylabel(month, fontsize=10)
#     ax.set_yticks([])  
    
#     if i < len(df.columns) - 1:
#         ax.set_xlabel('')
#     else:
#         ax.set_xlabel('Energia líquida gerada', fontsize=12)

# plt.tight_layout()
# plt.savefig('grafico_violin.png', dpi=700, bbox_inches='tight')
# plt.show()

# ### UGMT ###

# csv_path_ugmt = r'C:\TCC_V02\BDGD_2023\UGMT_tab.csv'

# UGMT = pd.read_csv(csv_path_ugmt, sep=';', encoding='latin1')

# colunas_energia = [f'ENE_{str(i).zfill(2)}' for i in range(1, 13)]

# dados_UGMT = UGMT[colunas_energia].copy()

# for col in colunas_energia:
#     dados_UGMT[col] = (dados_UGMT[col].astype(str).str.replace(',', '.', regex=False))
#     dados_UGMT = dados_UGMT.apply(pd.to_numeric, errors='coerce')

# dados_sem_zero_ugmt = dados_UGMT.mask(dados_UGMT = 0)

# ugmt_mean_mensal = dados_sem_zero_ugmt.mean(skipna=True)
# ugmt_max_mensal = dados_sem_zero_ugmt.max(skipna=True)

# valores_ugmt = dados_sem_zero_ugmt.values.flatten()
# valores_ugmt = valores_ugmt[~np.isnan(valores_ugmt)]

# q1_ugmt = np.quantile(valores_ugmt, 0.25)
# median_ugmt = np.median(valores_ugmt)
# q3_ugmt = np.quantile(valores_ugmt, 0.75)
# max_ugmt = np.max(valores_ugmt)
# mean_ugmt = np.mean(valores_ugmt)

# # Estatísticas por consumidor #
# UGMT['media_consumidor'] = dados_sem_zero_ugmt.mean(axis=1, skipna=True)
# UGMT['max_consumidor'] = dados_sem_zero_ugmt.max(axis=1, skipna=True)

# # Caminho para salvar txt com os resultados de media e maximo #
# saida_txt_ugmt = os.path.join(os.path.dirname(csv_path_ugmt), "resultados_ugmt.txt")

# with open(saida_txt_ugmt, "w", encoding="utf-8") as f:

#     f.write("UGMT")
#     f.write(f"Primeiro Quartil (Q1): {q1_ugmt:.2f} kWh")
#     f.write(f"Mediana (Q2): {median_ugmt:.2f} kWh")
#     f.write(f"Terceiro Quartil (Q3): {q3_ugmt:.2f} kWh")
#     f.write(f"Valor máximo: {max_ugmt:.2f} kWh")
#     f.write(f"Média: {mean_ugmt:.2f} kWh")

#     f.write("Média mensal:")
#     for col, val in ugmt_mean_mensal.items():
#         f.write(f"{col}: {val:.2f} kWh")

#     f.write("Valor máximo mensal:")
#     for col, val in ugmt_max_mes.items():
#         f.write(f"{col}: {val:.2f} kWh")

# print(f"Arquivo salvo: {saida_txt_ugmt}")

# ### SSDBT ###

# csv_path_ssdbt = r'C:\TCC_V02\BDGD_2023\SSDBT.csv'

# SSDBT = pd.read_csv(csv_path_ssdbt, sep=';')

# feeders = sorted(SSDBT["UNI_TR_MT"].unique())
# filtered_SSDBT = SSDBT[SSDBT["UNI_TR_MT"].isin(feeders)]

# grouped = filtered_SSDBT.groupby('UNI_TR_MT')['COMP'].sum().reset_index()
# grouped.columns = ['UNI_TR_MT', 'total_COMP']

# df = pd.DataFrame(grouped['total_COMP'], columns=['total_COMP'])

# # Criar gráfico #
# fig, ax = plt.subplots(figsize=(6, 3))
# sns.violinplot(x='total_COMP', data=df, scale='area', inner=None, cut=0, bw=0.15, width=0.6, ax=ax, palette='Blues')

# y = np.random.uniform(low=-0.04, high=0.04, size=len(df))
# ax.scatter(df['total_COMP'], y, color='blue', alpha=0.15, s=12)

# ax.boxplot(df['total_COMP'], vert=False, positions=[0], widths=0.12, showfliers=False, medianprops={'color': 'red', 'linewidth': 2}, boxprops={'linewidth': 1.2},whiskerprops={'linewidth': 1.2})
# ax.set_xlim(0, 3000)
# ax.set_yticks([])
# ax.set_xlabel("Comprimento Total [m]", fontsize=10, labelpad=8)
# plt.tight_layout()
# plt.show()

# # Idicadores calculados #
# q1 = df['total_COMP'].quantile(0.25)
# median = df['total_COMP'].median()
# q3 = df['total_COMP'].quantile(0.75)
# max_val = df['total_COMP'].max()
# mean_val = df['total_COMP'].mean()

# # Caminho para salvar os resultados em arquivo .txt #
# saida_txt_ssdbt = os.path.join(os.path.dirname(csv_path_ssdbt), "estatisticas_ssdbt.txt")

# # Salvar estatísticas no .txt #
# with open(saida_txt_ssdbt, "w", encoding="utf-8") as f:
#     f.write("SSDBT")
#     f.write(f"Primeiro Quartil (Q1): {q1:.2f} m")
#     f.write(f"Mediana (Q2): {median:.2f} m")
#     f.write(f"Terceiro Quartil (Q3): {q3:.2f} m")
#     f.write(f"Valor máximo: {max_val:.2f} m")
#     f.write(f"Média: {mean_val:.2f} m")

# print(f"Arquivo SSDBT salvo em: {saida_txt_ssdbt}")

# ### SSDMT ###

# csv_path_ssdmt = r'C:\TCC_V02\BDGD_2023\SSDMT.csv'

# SSDMT = pd.read_csv(csv_path_ssdmt, sep=';')

# feeders = sorted(SSDMT["CTMT"].unique())
# filtered_SSDMT = SSDMT[SSDMT["CTMT"].isin(feeders)]

# grouped = filtered_SSDMT.groupby('CTMT')['COMP'].sum().reset_index()
# grouped.columns = ['feeder', 'total_COMP']

# df = pd.DataFrame(grouped['total_COMP'], columns=['total_COMP'])

# # Criar grafico #
# fig, ax = plt.subplots(figsize=(6, 3))
# sns.violinplot(x='total_COMP', data=df, scale='area', inner=None, cut=0, bw=0.15, width=0.6, ax=ax, palette='Blues')

# y = np.random.uniform(low=-0.04, high=0.04, size=len(df))
# ax.scatter(df['total_COMP'], y, color='blue', alpha=0.5, s=12)

# ax.boxplot(df['total_COMP'], vert=False, positions=[0], widths=0.12, showfliers=False, medianprops={'color': 'red', 'linewidth': 2}, boxprops={'linewidth': 1.2}, whiskerprops={'linewidth': 1.2})

# ax.set_xlim(0, 700000)
# ax.set_yticks([])
# ax.set_xlabel("Comprimento Total [m]", fontsize=10, labelpad=8)
# plt.tight_layout()
# plt.show()

# # Calcular dados # 
# q1 = df['total_COMP'].quantile(0.25)
# median = df['total_COMP'].median()
# q3 = df['total_COMP'].quantile(0.75)
# max_val = df['total_COMP'].max()
# mean_val = df['total_COMP'].mean()

# # Caminho para salvar o arquivo .txt com os resultados #
# saida_txt_ssdmt = os.path.join(os.path.dirname(csv_path_ssdmt), "estatisticas_ssdmt.txt")

# # Salvar no .txt
# with open(saida_txt_ssdmt, "w", encoding="utf-8") as f:
#     f.write("Resultados - SSDMT")
#     f.write(f"Primeiro Quartil (Q1): {q1:.2f} m")
#     f.write(f"Mediana (Q2): {median:.2f} m")
#     f.write(f"Terceiro Quartil (Q3): {q3:.2f} m")
#     f.write(f"Valor máximo: {max_val:.2f} m")
#     f.write(f"Média: {mean_val:.2f} m")

# print(f"Arquivo SSDMT salvo em: {saida_txt_ssdmt}")

# ### TRANSFORMADORES ###

# csv_path_transf = r'C:\TCC_V02\BDGD_2023\UNTRMT.csv'

# TRANSF = pd.read_csv(csv_path_transf, sep=';', encoding='latin1')

# TRANSF['POT_NOM'] = pd.to_numeric(TRANSF['POT_NOM'], errors='coerce')

# media_pot = TRANSF['POT_NOM'].mean()
# max_pot = TRANSF['POT_NOM'].max()
# min_pot = TRANSF['POT_NOM'].min()
# moda_pot = TRANSF['POT_NOM'].mode().iloc[0] if not TRANSF['POT_NOM'].mode().empty else None

# print("Transformadores")
# print(f"Média: {media_pot:.2f} kVA")
# print(f"Valor máximo: {max_pot:.2f} kVA")
# print(f"Moda: {moda_pot:.2f} kVA")

# saida_txt_transf = os.path.join(os.path.dirname(csv_path_transf), "resultados_transformadores.txt")

# with open(saida_txt_transf, "w", encoding="utf-8") as f:
#     f.write(f"Média: {media_pot:.2f} kVA")
#     f.write(f"Valor máximo: {max_pot:.2f} kVA")
#     f.write(f"Moda: {moda_pot:.2f} kVA")

# print(f"nArquivo salvo em: {saida_txt_transf}")
