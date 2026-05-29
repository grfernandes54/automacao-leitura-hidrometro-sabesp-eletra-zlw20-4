# Calibração — quantos litros vale 1 pulso?

Este é o passo que separa um sensor que funciona de um que mente. **O fator de calibração (litros por pulso) muda de medidor para medidor**, e errar nele faz o consumo medido divergir do real em dezenas por cento.

---

## ⚠️ Por que NÃO confiar no "R500" da etiqueta

A etiqueta do Eletra ZLW20-4 traz `R500 (H/V)`. É tentador concluir "R500 = 500 pulsos por m³" e sair multiplicando. **Isso está errado.**

`R` (às vezes escrito como a razão Q3/Q1) é uma classe metrológica que descreve a **faixa de medição** do hidrômetro — o quão bem ele mede vazões muito baixas em relação às altas. **Não tem nada a ver com o peso do pulso elétrico.**

> 💡 **A armadilha em números:** assumindo erroneamente 500 pulsos/m³ (= 2 L/pulso), o projeto registrava só uma fração da água real. A calibração contra o display revelou o verdadeiro fator: **1 pulso = 10 L** (100 pulsos/m³). Ou seja, o medidor pulsava 5× menos do que a etiqueta sugeria — e ~80% do consumo "sumia". **Só a medição contra o display revela o fator real.**

No Eletra ZLW20-4 / Zlink da Sabesp, validado em campo:

| Medido | Valor |
|--------|-------|
| 1 pulso | **0,01 m³ = 10 L** |
| 100 pulsos | 1 m³ |
| Conferência | 16 pulsos ↔ 0,159 m³ no display → 159 L ÷ 16 = **9,94 L/pulso ≈ 10** |

---

## Procedimento de calibração (faça no seu medidor)

1. Anote a leitura do **display do hidrômetro** (precisão de 1 L). → `display_inicio`
2. Anote os **"Pulsos Totais"** da ESP no mesmo instante. → `pulsos_inicio`
   - Dica: o log do ESPHome (nível DEBUG) registra o volume a cada pulso; ou observe o `Volume Acumulado` e divida pelo `multiply` atual para obter pulsos.
3. **Use um volume conhecido de água** — entre ~100 e 500 L. Encher uma caixa-d'água ou um tambor ajuda a chegar a um número grande (quanto maior o volume, menor o erro relativo).
4. Anote o display e os pulsos no fim. → `display_fim`, `pulsos_fim`
5. Calcule:

   ```
   fator (L/pulso) = (display_fim − display_inicio) × 1000
                     ─────────────────────────────────────
                          (pulsos_fim − pulsos_inicio)
   ```

   *(× 1000 converte m³ do display para litros)*

6. Para o Zlink/Sabesp o resultado deve ficar perto de **10 L/pulso**. Outros medidores podem dar **1, 5 ou 100 L/pulso** — daí a importância de medir o seu.

---

## Como ajustar o YAML

O firmware usa dois multiplicadores em [`esphome/medidor-agua.yaml`](../esphome/medidor-agua.yaml):

```yaml
# Vazão (pulsos/min → m³/h):  multiply = (L/pulso ÷ 1000) × 60
filters:
  - multiply: 0.6     # para 10 L/pulso

# Total (pulsos → m³):  multiply = L/pulso ÷ 1000
total:
  filters:
    - multiply: 0.01  # para 10 L/pulso
```

Regra geral, conforme o seu fator:

| L/pulso | `multiply` da vazão | `multiply` do total |
|---------|---------------------|---------------------|
| 1 L | 0,06 | 0,001 |
| 5 L | 0,30 | 0,005 |
| **10 L** | **0,60** | **0,01** |
| 100 L | 6,0 | 0,1 |

Fórmulas: vazão = `(L/pulso ÷ 1000) × 60` · total = `L/pulso ÷ 1000`.

---

## Tabela para você preencher

| Campo | Valor |
|-------|-------|
| `display_inicio` (m³) | __________ |
| `pulsos_inicio` | __________ |
| `display_fim` (m³) | __________ |
| `pulsos_fim` | __________ |
| Volume real = (display_fim − display_inicio) × 1000 (L) | __________ |
| Pulsos contados = pulsos_fim − pulsos_inicio | __________ |
| **Fator = Volume real ÷ Pulsos contados (L/pulso)** | __________ |
| `multiply` vazão = (fator ÷ 1000) × 60 | __________ |
| `multiply` total = fator ÷ 1000 | __________ |

---

➡️ Sinal não bate ou não conta? Veja **[DIAGNOSTICO.md](DIAGNOSTICO.md)**.
