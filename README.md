# Estagio-ABAP-Relatorio

###RELATÓRIO ABAP

```abap

REPORT ZPROG_EX7_02.

*Fazer um relatório com a seguinte saída:
*
*ZPRODUTO_00 -> PRODUTO
*ZPRODUTO_00 -> DESCRIÇÃO
*ZPRODUTO_00 -> PREÇO
*ZESTOQUE_00 -> ESTOQUE
*CALCULADO   -> VALOR TOTAL DO PRODUTO EM ESTOQUE (QUANT * VALOR)
*ZVENDAS_00  -> QUANTIDADE
*ZVENDAS_00  -> VALOR DA VENDA
*ZVENDAS_00  -> DATA
*ZVENDAS_00  -> HORA
*
** Da tabela de vendas considerar a maior venda (venda com maior quantidade)
*
*
*00000001 Coca Colca             5,90      10      59,00   -- Nenhuma venda registrada --
*00000002 Presunto               2,50       5      12,50   10   25,00   16/10/2023  16:55
*00000003 Queijo                 3,80       0       0,00   -- Nenhuma venda registrada --

TABLES: zprodutos_02,
        zestoque_02,
        zvendas_02.

TYPES: BEGIN OF ty_estoque,
         produto    TYPE zproduto_02,
         quantidade TYPE zquantidade_02,
         unidade    TYPE zunidade_02,
       END OF ty_estoque.

DATA: lt_estoque TYPE TABLE OF ty_estoque,
      ls_estoque LIKE LINE OF lt_estoque,
      ls_produto TYPE zprodutos_e_02,
      lt_produto TYPE TABLE OF zprodutos_e_02,
      lt_vendas  TYPE TABLE OF zvendas_02,
      ls_venda   LIKE LINE OF lt_vendas.

DATA: lv_maior  TYPE zquantidade_02,
      lv_vlrtot TYPE zpreco_02,
      lv_maxidx TYPE sy-tabix.

SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-001.

SELECT-OPTIONS: s_prod FOR zprodutos_02-produto,
                s_prec FOR zprodutos_02-preco NO INTERVALS.

SELECTION-SCREEN END OF BLOCK b1.

START-OF-SELECTION.
  "Seleção de banco de dados para uma tabela interna
  SELECT produto
         desc_produto
         preco
    FROM zprodutos_02
    INTO CORRESPONDING FIELDS OF TABLE lt_produto
    WHERE produto IN s_prod
      AND preco IN s_prec.

  SELECT produto
         unidade
         quantidade
    FROM zestoque_02
    INTO TABLE lt_estoque
    FOR ALL ENTRIES IN lt_produto
    WHERE produto = lt_produto-produto.

  SELECT venda
         item
         produto
         hora
         preco
         unidade
         moeda
         data
         quantidade
    FROM zvendas_02
    INTO CORRESPONDING FIELDS OF TABLE lt_vendas
     FOR ALL ENTRIES IN lt_produto
    WHERE produto = lt_produto-produto.


END-OF-SELECTION.

  LOOP AT lt_produto INTO ls_produto.

    "Busca o produto
    READ TABLE lt_estoque INTO ls_estoque WITH KEY produto = ls_produto-produto.
    IF sy-subrc <> 0.
      ls_estoque-quantidade = 0.
    ENDIF.

    "Valor total do estoque
    lv_vlrtot = ls_produto-preco * ls_estoque-quantidade.

    "Busca maior venda do produto
    CLEAR: ls_venda, lv_maior, lv_maxidx.
    LOOP AT lt_vendas INTO ls_venda WHERE produto = ls_produto-produto.
      IF ls_venda-quantidade > lv_maior.
        lv_maior  = ls_venda-quantidade.
        lv_maxidx = sy-tabix.
      ENDIF.
    ENDLOOP.

    WRITE: / ls_produto-produto, ls_produto-descricao, ls_produto-preco, ls_estoque-quantidade, lv_vlrtot.
    IF ls_venda-produto = ''.
      WRITE: '-- Nenhuma venda registrada --'.
    ELSE.
      READ TABLE lt_vendas INTO ls_venda INDEX lv_maxidx.
      WRITE: ls_venda-quantidade, ls_venda-preco, ls_venda-data, ls_venda-hora.
    ENDIF.

  ENDLOOP.

```
