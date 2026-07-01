##  Detalhes Técnicos e Justificativas de Design

###  Stopwords específicas do domínio acadêmico

São stopwords específicas do domínio acadêmico: palavras que aparecem em praticamente todo artigo científico, independente do tema. Se não forem removidas, elas teriam o TF-IDF inflado, bagunçando e criando viés no modelo.

---

###  Transferência de Aprendizado (Transfer Learning)

O `pt_core_news_sm` é um modelo pré-treinado para tarefas como tokenização, lematização, POS-tagging e reconhecimento de entidades. Ao carregá-lo, o pipeline reaproveita conhecimento linguístico geral do português (morfologia, flexões verbais, concordância) sem precisar treinar nada do zero.

---

###  Dicionário Curado de Domínio

Em vez de deixar o algoritmo "adivinhar" os temas dos clusters puramente por estatística (TF-IDF), é aplicado ao pipeline um dicionário curado de domínio para mapear termos → área técnica.

---

###  Dois Espaços Vetoriais, Dois Propósitos

O código usa dois espaços vetoriais diferentes para propósitos diferentes:

- **`X_cluster`** → usado para agrupar os documentos
- **`X_tfidf`** → usado para explicar os grupos via termos interpretáveis

Essa separação é uma escolha metodologicamente correta: embeddings densos capturam melhor a semântica para fins de clustering, enquanto o TF-IDF é ruim para medir similaridade semântica profunda, mas excelente para extrair palavras-chave interpretáveis.

---

###  Ensemble de Métricas para Escolha do Modelo

Nenhuma das métricas de validação é confiável isoladamente (cada uma possui seu próprio viés). Por isso, o código converte cada métrica em ranking e soma os ranks, escolhendo a configuração com a menor soma. Isso é, essencialmente, um método de votação/ensemble de métricas, que reduz o viés de qualquer métrica individual.

---

###  Sobre o Gráfico

**Mediana em vez de média:** foi usada a mediana, em vez da média, para posicionar os rótulos no gráfico. Essa é uma escolha deliberada e correta — a média é sensível a outliers (pontos "perdidos" longe do núcleo do cluster puxariam o centro para fora da massa visual real).

Como o UMAP frequentemente produz clusters com formas alongadas e não-convexas, a proximidade visual entre clusters sugere similaridade semântica, mas isso deve ser entendido como uma **hipótese visual**, não como uma prova estatística.

---

###  O Caso "Agro" (duas vezes)

O rótulo **"Agro"** aparece duas vezes no gráfico, mesmo recebendo o mesmo rótulo textual. Isso indica que o algoritmo de clustering encontrou uma distinção real nos dados que o sistema de nomeação por radical não conseguiu capturar (ex: *agricultura de precisão* vs. *agroindústria/biológico*) — os dois clusters são semanticamente distintos o suficiente para o modelo separá-los, mas compartilham o mesmo vocabulário técnico de superfície.

O que poderia ser feito aqui seria, na parte do "transfer learning", criar mais um tema com palavras mais específicas, já que apenas "Agro" ficou abrangente demais.

> 💡💡 Achei interessante levantar esse ponto, pois ele mostra uma limitação dos modelos em geral — então deixei assim mesmo, como registro dessa observação.💡💡
