# Roadmap - Transcritor PDF

Este documento detalha as fases e tarefas planejadas para o desenvolvimento do projeto `transcritor-pdf`. Usaremos checkboxes (`[ ]` pendente, `[x]` concluído) para acompanhar o progresso. **Nota:** Requisitos atualizados em 24/04/2025 para incluir processamento página-a-página, objetivo final de RAG, e interação com Vector DB.

**Fase 0: Configuração Inicial e Planejamento**

* [x] Definição do Escopo e Objetivos do Projeto (Atualizado: Saída para RAG + Vetorização)
* [x] Definição das Tecnologias Principais (Python, Langchain, Docling, OpenRouter)
* [x] Definição da Arquitetura Modular (src/input, preprocess, extract, output, vectorizer)
* [x] Criação da Estrutura Inicial de Pastas e Arquivos (`src/`, `main.py`, `__init__.py`)
* [x] Criação do `README.md` inicial
* [x] Criação do `requirements.txt` inicial (`langchain`, `openai`, `python-dotenv`)
* [x] Configuração do Repositório Git e Conexão com GitHub (`galvani4987/transcritor-pdf`)
* [x] Criação e Ativação do Ambiente Virtual (`.venv`)
* [x] Instalação das Dependências Iniciais (`pip install -r requirements.txt`)
* [x] Configuração do `.env` com a chave da API OpenRouter (feito pelo usuário)
* [x] Criação deste arquivo `ROADMAP.md`
* [x] Realizar o commit das atualizações no `ROADMAP.md` e arquivos de configuração.

**Fase 1: Tratamento da Entrada (Módulo `src/input_handler`)**

* [x] Adicionar dependência para manipulação de PDF (ex: `pypdfium2`) ao `requirements.txt` e instalar.
* [x] Implementar análise básica de argumentos de linha de comando (CLI) em `src/main.py` usando `argparse` para receber o caminho do arquivo PDF de entrada.
* [x] Criar `src/input_handler/pdf_splitter.py`.
* [x] **Implementar função em `pdf_splitter.py` para dividir um arquivo PDF de entrada em páginas individuais (salvando como imagens temporárias WebP em disco).**
* [x] Criar `src/input_handler/loader.py`.
* [x] Implementar função em `loader.py` para carregar uma imagem de página (a partir do caminho do arquivo temporário). Validar o input.
* [x] Implementar tratamento de erros para caminhos inválidos, PDFs corrompidos ou tipos de arquivo não suportados. (Validação inicial e por página implementadas).
* [x] Modificar o fluxo principal em `src/main.py` para:
    * Receber o caminho do PDF.
    * Chamar o `pdf_splitter` para obter os caminhos das páginas.
    * **Iterar sobre cada caminho de página**, chamando as fases seguintes (placeholders) para cada uma.
* [x] Realizar commit das funcionalidades de tratamento de entrada e divisão de PDF.

**Fase 2: Pré-processamento por Página (Módulo `src/preprocessor`)**

* [x] Adicionar dependências de pré-processamento ao `requirements.txt` (ex: `pillow`) e instalar.
* [x] Criar `src/preprocessor/image_processor.py`.
* [x] Pesquisar e decidir sobre técnicas de pré-processamento de imagem para manuscritos. **(Concluído: Pesquisa indicou pipeline com Scikit-image: Deskew -> Grayscale -> Median -> CLAHE -> Sauvola)**
* [x] Adicionar dependência `scikit-image` ao `requirements.txt` e instalar.
* [x] Implementar **Deskewing** (Correção de Inclinação) em `image_processor.py`.
* [x] Implementar **Conversão para Escala de Cinza** em `image_processor.py`.
* [x] Implementar **Filtro de Mediana** (Redução de Ruído) em `image_processor.py` usando Scikit-image.
* [x] Implementar **CLAHE** (Melhora de Contraste) em `image_processor.py` usando Scikit-image.
* [x] Implementar **Binarização Sauvola** em `image_processor.py` usando Scikit-image.
* [x] Implementar [Opcional] **Recorte de Bordas** em `image_processor.py`.
* [x] Integrar a chamada à função `preprocess_image` no loop de página em `src/main.py`.
* [x] Realizar commit das funcionalidades de pré-processamento.
* [ ] _(Pendente)_ Pesquisar como usar `Docling` (ou similar) para análise de layout **em cada imagem de página** (detectar blocos de texto, áreas de assinatura).
* [ ] _(Pendente)_ Criar `src/preprocessor/layout_analyzer.py` (ou integrar).
* [ ] _(Pendente)_ Implementar a integração com `Docling` para obter informações de layout **por página**.

**Fase 3: Extração de Informações por Página (Módulo `src/extractor`)**

* [x] Criar `src/extractor/llm_client.py`.
* [x] Implementar a lógica para carregar a API Key do OpenRouter do `.env`.
* [x] Implementar a configuração e inicialização do cliente Langchain (OpenRouter).
* [x] Criar `src/extractor/text_extractor.py`.
* [x] Desenvolver prompt/cadeia Langchain para extrair texto (OCR) **da imagem da página pré-processada**.
* [x] Criar `src/extractor/info_parser.py`.
* [x] Desenvolver prompt/cadeia Langchain para analisar o texto extraído **da página** e identificar/extrair informações chave (JSON Output).
* [x] Refinar prompts para precisão e custo **por página** (tanto para text_extractor quanto info_parser). (Refinamento inicial concluído).
* [x] Implementar tratamento de erros para chamadas à API LLM (mais robusto, com retentativas).
* [x] Integrar as etapas de extração (text_extractor) no loop de página em `src/main.py`.
* [x] Integrar a chamada ao `info_parser` no loop de página em `src/main.py`.
* [x] Realizar commit das funcionalidades de extração por página.

**Fase 4: Tratamento da Saída para RAG (Módulo `src/output_handler`)**

* [x] Criar `src/output_handler/formatter.py`.
* [x] **Definir formato de saída estruturado (ex: lista de JSONs, um por página/chunk) otimizado para RAG.** (Função inicial criada, definição pode ser refinada).
* [x] Implementar função em `formatter.py` para gerar essa saída estruturada a partir dos dados extraídos de cada página. (Função inicial criada, chunking básico implementado).
* [x] Implementar a exibição (talvez resumida) do resultado no console.
* [x] Implementar a opção de salvar a saída estruturada completa em um arquivo (ex: `.jsonl`).
* [x] Integrar o tratamento de saída no final do processamento do PDF em `src/main.py`.
* [x] Realizar commit das funcionalidades de tratamento de saída para RAG.

**Fase 5: Vetorização e Armazenamento (Módulo `src/vectorizer`)**

* [x] Escolher biblioteca e modelo de embedding: **OpenAI `text-embedding-3-small` via API**. Adicionar dependência `langchain-openai` ao `requirements.txt` e instalar.
* [x] Escolher Vector DB: **PostgreSQL com extensão vetorial** (pré-existente no `modular-dashboard`). Adicionar dependência `asyncpg` (+ `psycopg2-binary` se necessário para compatibilidade) ao `requirements.txt` e instalar.
* [x] Criar módulo `src/vectorizer/` com `__init__.py`.
* [x] Criar `src/vectorizer/embedding_generator.py`.
* [x] Implementar lógica para gerar embeddings (usando OpenAI) para os chunks de texto da saída formatada (Fase 4).
* [x] Criar `src/vectorizer/vector_store_handler.py`.
* [x] Implementar lógica para inicializar/conectar ao PostgreSQL.
* [x] Implementar lógica para adicionar/atualizar os vetores e metadados na tabela PostgreSQL apropriada.
* [x] Integrar a etapa de vetorização no fluxo principal (`main.py`), após a Fase 4.
* [x] Realizar commit da funcionalidade de vetorização.

**Fase 6: Integração Final, Testes e Refinamento**

* [x] Revisar a integração de todos os módulos no pipeline principal (loop por página + vetorização).
* [x] Adicionar logging adequado.
* [x] Criar a estrutura da pasta `tests/`.
* [x] Escrever testes unitários e de integração (incluindo vetorização). (Testes unitários com mocks para módulos principais concluídos).
* [x] Refatorar código (clareza, eficiência, manutenibilidade). (Refatorado vector_store_handler para asyncpg, main loop para helper).
* [x] Adicionar/Melhorar docstrings. (Módulos principais concluídos).
* [x] Atualizar `README.md` com instruções de uso completas.
* [x] Realizar commit da versão final do CLI.

**Fase 7: Futuro - Integração com `modular-dashboard`**

* [ ] Analisar requisitos para integração.
* [ ] Definir API/interface programática.
* [ ] Refatorar código para ser reutilizável como biblioteca.
* [ ] Implementar a interface de integração.