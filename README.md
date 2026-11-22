# Aplicativo Entregador

App Android (Java) de apoio para controle de entregas, ganhos e metas.

## Como abrir

1. Abra o **Android Studio**
2. `File > Open` → selecione a pasta `AplicativoEntregador`
3. Aguarde o Gradle sincronizar (primeira vez pode demorar um pouco, baixando dependências)
4. Rode em um emulador ou celular conectado (▶ Run)

> Este projeto não inclui o `gradlew` (wrapper do Gradle) pronto — o Android
> Studio detecta isso automaticamente e oferece pra gerar o wrapper na hora de
> abrir o projeto. É só aceitar.

## O que já está funcionando

- **Início**: ganhos do dia, corridas, km rodado, R$/km, status da meta do mês, últimas corridas (com **checkbox pra excluir**), **controle de turno com Iniciar / Parar / Retomar / Zerar**
- **Nova corrida**: registro manual (valor, distância, tipo de veículo) salvo no banco local (Room), com máscara de moeda e de km enquanto digita
- **Veículo**: registro de **gastos** (combustível, manutenção, IPVA) e **lembretes de manutenção**, ambos com **checkbox + botão Excluir**
- **Metas**: meta mensal configurável; saldo negativo (vermelho) até bater a meta, positivo (verde) depois
- **Relatórios**: gráfico dos últimos 7 dias, lucro líquido já descontando os gastos reais, exportação em **XLSX** (Excel de verdade, sem precisar do CSV)
- **Modo claro/escuro**: automático, segue o tema do sistema (Android `DayNight`); botão de alternar também disponível
- **Navegação por arrastar**: as 5 telas usam `ViewPager2` — dá pra arrastar pro lado (esquerda/direita) além de tocar nos ícones da barra inferior

## Como funciona o modo Turno

Na tela Início tem um cronômetro com os botões:

- **Iniciar** — começa a contar
- **Parar** — pausa o cronômetro, mas **não zera** o tempo
- **Retomar** — volta a contar de onde parou (soma com o tempo anterior)
- **Zerar** — só aparece quando está pausado; aí sim zera o tempo de vez

Isso cobre o caso comum de pausar pro almoço e continuar depois sem perder o
tempo já trabalhado. Cada Iniciar→Parar fica salvo como uma sessão na tabela
`sessoes_turno`, pronta pra uso em relatórios futuros. Se você fechar o app
com o turno rodando, ao reabrir o cronômetro continua certinho (o cálculo usa
o horário salvo, não depende do app ficar aberto).

## Como funcionam os lembretes de manutenção

Você cadastra um nome (ex: "Troca de óleo") e um intervalo em km (ex: 3000).
O app soma automaticamente os km de todas as corridas registradas desde a
última vez que você marcou aquele item como "Feito", e mostra o progresso
numa barrinha — fica vermelho quando passa do intervalo definido.

> Importante: como o progresso depende do km das corridas que você registra,
> a precisão dele depende de você preencher o campo "distância" ao salvar
> cada corrida.

## Como excluir itens

Em **Início** (últimas corridas), **Veículo → Gastos** e **Veículo →
Manutenção**, cada item tem um checkbox. Marca o(s) que quiser remover e toca
no botão **Excluir** que fica ao lado do título da seção. Pode marcar vários
de uma vez.

## O que ainda pode evoluir (ideias pra depois)

- Editar (não só excluir) gastos e lembretes de manutenção já cadastrados
- Histórico completo de turnos na tela de Relatórios
- Gráfico de gastos por categoria

## Estrutura do projeto

```
app/src/main/java/com/entregador/aplicativo/
├── AtividadePrincipal.java            → ViewPager2 + barra de navegação sincronizados
├── AdaptadorPaginasPrincipal.java     → alimenta o ViewPager2 com as 5 telas
├── dados/
│   ├── Corrida.java                   → entidade (uma corrida)
│   ├── CorridaDao.java                → consultas ao banco (inclui excluirPorIds)
│   ├── Gasto.java                     → entidade (um gasto)
│   ├── GastoDao.java
│   ├── ItemManutencao.java            → entidade (um lembrete de manutenção)
│   ├── ManutencaoDao.java
│   ├── SessaoTurno.java               → entidade (um turno de trabalho)
│   ├── SessaoTurnoDao.java
│   ├── BancoDeDadosApp.java           → configuração do Room
│   └── Preferencias.java              → meta mensal, turno ativo e tempo acumulado
├── telas/inicio/                      → FragmentoInicio + AdaptadorCorridas (turno + últimas corridas)
├── telas/novacorrida/                 → FragmentoNovaCorrida
├── telas/veiculo/                     → FragmentoVeiculo + AdaptadorGastos (gastos e manutenção)
├── telas/meta/                        → FragmentoMeta + AjudantePesoLinearLayout
├── telas/relatorios/                  → FragmentoRelatorios (exportação XLSX)
└── utilitarios/                       → UtilitariosData, UtilitariosPeso, ExportadorXlsx,
                                         ObservadorTextoMoeda, ObservadorTextoDecimal
```

Banco de dados: **Room (SQLite)**, tudo local no celular — nenhum dado sai do
aparelho, nenhuma dependência de servidor.
