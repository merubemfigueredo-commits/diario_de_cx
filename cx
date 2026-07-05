import streamlit as st
import pandas as pd
from datetime import date, datetime

st.set_page_config(
    page_title="Controle Diário de Caixa",
    page_icon="💰",
    layout="wide",

st.title("💰 Controle Diário de Caixa")
st.caption("Registre entradas e saídas e acompanhe o saldo a cada movimentação.")

# ── Estado da sessão ──────────────────────────────────────────────────────────
if "movimentacoes" not in st.session_state:
    st.session_state.movimentacoes = []

CATEGORIAS_ENTRADA = [
    "Venda de Produtos", "Prestação de Serviços", "Recebimento de Clientes",
    "Rendimentos Financeiros", "Outras Entradas",
]
CATEGORIAS_SAIDA = [
    "Compras / Fornecedores", "Salários e Encargos", "Impostos e Taxas",
    "Aluguel / Utilidades", "Marketing e Publicidade", "Equipamentos", "Outras Saídas",
]

def fmt(v: float) -> str:
    return f"R$ {v:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")

# ── Layout: formulário | extrato ───────────────────────────────────────────────
col_form, col_extrato = st.columns([1, 2])

# ── Formulário de nova movimentação ───────────────────────────────────────────
with col_form:
    st.subheader("Nova Movimentação")
    with st.form("form_mov", clear_on_submit=True):
        data_mov  = st.date_input("Data", value=date.today())
        tipo      = st.radio("Tipo", ["Entrada", "Saída"], horizontal=True)
        cats      = CATEGORIAS_ENTRADA if tipo == "Entrada" else CATEGORIAS_SAIDA
        categoria = st.selectbox("Categoria", cats)
        descricao = st.text_input("Descrição", placeholder="Ex: Venda balcão #142")
        valor     = st.number_input("Valor (R$)", min_value=0.01, step=0.01, format="%.2f")
        submit    = st.form_submit_button("✅ Adicionar", use_container_width=True)

    if submit:
        if not descricao.strip():
            st.error("Informe uma descrição.")
        else:
            st.session_state.movimentacoes.append({
                "data":      data_mov.isoformat(),
                "descricao": descricao.strip(),
                "categoria": categoria,
                "tipo":      tipo,
                "valor":     valor if tipo == "Entrada" else -valor,
            })
            st.success(f"{tipo} de {fmt(valor)} registrada!")
            st.rerun()

# ── Extrato do dia selecionado ─────────────────────────────────────────────────
with col_extrato:
    st.subheader("Extrato do Dia")

    datas_com_movs = sorted(
        {m["data"] for m in st.session_state.movimentacoes}, reverse=True
    )
    data_filtro_str = st.date_input(
        "Selecionar data",
        value=date.today(),
        key="filtro_data",
        label_visibility="collapsed",
    ).isoformat()

    movs = st.session_state.movimentacoes
    movs_ate_ontem = [m for m in movs if m["data"] < data_filtro_str]
    movs_do_dia    = [m for m in movs if m["data"] == data_filtro_str]

    saldo_anterior = sum(m["valor"] for m in movs_ate_ontem)

    # Monta tabela com saldo corrente
    saldo = saldo_anterior
    linhas = []
    for m in movs_do_dia:
        saldo += m["valor"]
        linhas.append({
            "Descrição":  m["descricao"],
            "Categoria":  m["categoria"],
            "Entradas":   fmt(m["valor"])  if m["valor"] >= 0 else "—",
            "Saídas":     fmt(-m["valor"]) if m["valor"] < 0  else "—",
            "Saldo":      fmt(saldo),
            "_tipo":      m["tipo"],
            "_saldo_num": saldo,
        })

    total_ent = sum(m["valor"] for m in movs_do_dia if m["valor"] >= 0)
    total_sai = sum(-m["valor"] for m in movs_do_dia if m["valor"] < 0)
    resultado = total_ent - total_sai
    saldo_final = saldo

    # KPIs do dia
    k1, k2, k3, k4 = st.columns(4)
    k1.metric("Saldo Anterior", fmt(saldo_anterior))
    k2.metric("Entradas", fmt(total_ent), delta=fmt(total_ent) if total_ent else None, delta_color="normal")
    k3.metric("Saídas", fmt(total_sai), delta=f"-{fmt(total_sai)}" if total_sai else None, delta_color="inverse")
    k4.metric("Saldo Final", fmt(saldo_final), delta=fmt(resultado) if resultado != 0 else None,
              delta_color="normal" if resultado >= 0 else "inverse")

    st.divider()

    if not linhas:
        st.info("Nenhuma movimentação nesta data. Use o formulário ao lado para adicionar.")
    else:
        df = pd.DataFrame(linhas)

        # Destaca entradas e saídas
        def colorir(row):
            if row["_tipo"] == "Entrada":
                return ["background-color: #f0fdf4"] * len(row)
            else:
                return ["background-color: #fff1f2"] * len(row)

        df_display = df[["Descrição", "Categoria", "Entradas", "Saídas", "Saldo"]]
        st.dataframe(
            df_display,
            use_container_width=True,
            hide_index=True,
            column_config={
                "Entradas": st.column_config.TextColumn(width="small"),
                "Saídas":   st.column_config.TextColumn(width="small"),
                "Saldo":    st.column_config.TextColumn(width="medium"),
            },
        )

        # Linha de totais
        st.markdown(
            f"""
            | | | **Entradas** | **Saídas** | **Saldo Final** |
            |---|---|---|---|---|
            | **Total** | | **{fmt(total_ent)}** | **{fmt(total_sai)}** | **{fmt(saldo_final)}** |
            """,
            unsafe_allow_html=False,
        )

# ── Excluir movimentações ──────────────────────────────────────────────────────
if st.session_state.movimentacoes:
    with st.expander("⚠️ Gerenciar / Excluir movimentações"):
        df_all = pd.DataFrame(st.session_state.movimentacoes)
        df_all["valor_fmt"] = df_all["valor"].apply(fmt)
        df_all_display = df_all[["data", "tipo", "descricao", "categoria", "valor_fmt"]].rename(columns={
            "data": "Data", "tipo": "Tipo", "descricao": "Descrição",
            "categoria": "Categoria", "valor_fmt": "Valor",
        })

        st.dataframe(df_all_display, use_container_width=True, hide_index=False)

        idx_excluir = st.number_input(
            "Índice da linha a excluir (ver tabela acima)", min_value=0,
            max_value=max(0, len(st.session_state.movimentacoes) - 1), step=1,
        )
        if st.button("🗑️ Excluir linha selecionada"):
            st.session_state.movimentacoes.pop(idx_excluir)
            st.success("Movimentação excluída.")
            st.rerun()

        if st.button("🗑️ Limpar TUDO", type="secondary"):
            st.session_state.movimentacoes = []
            st.rerun()

# pip install streamlit pandas
# streamlit run app_controle_diario.py

