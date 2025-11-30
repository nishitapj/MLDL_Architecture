# MLDL_Architecture
```mermaid
graph TD
    %% INPUT STAGE
    Input["Raw Legal Document\nBillSum: US/Cali Bills"] --> Pre["Text Preprocessing\n- Whitespace normalization\n- Lowercasing (optional)"]

    %% TRAINING PATH (LEFT)
    subgraph Training_Path["Model Training Path"]
        style Training_Path fill:#e6f3ff,stroke:#333,stroke-width:2px

        Pre --> T5Token["T5 Tokenizer\nPrefix: 'summarize:'"]
        T5Token --> Dataset["Custom BillSum Dataset\nMax Input = 320\nMax Target = 128"]

        Dataset --> T5Model["T5-small Seq2Seq Model\n(Encoder–Decoder)"]

        Optim["AdamW Optimizer"] -.-> T5Model
        Sched["Linear Scheduler\nwith Warmup + Decay"] -.-> T5Model
        GradAcc["Gradient Accumulation\n(Eff. BS = 8) + Clipping"] -.-> T5Model

        T5Model --> Decode["Beam Search Decoding\nBeam Width = 4"]
        Decode --> GenSummary["Generated Summary\n(Abstractive)"]
    end

    %% TESTING / STRUCTURAL PATH (RIGHT)
    subgraph Test_Path["Testing / Structural Verification Path"]
        style Test_Path fill:#e6ffec,stroke:#333,stroke-width:2px

        Pre --> SpaCy["spaCy Processing\n(en_core_web_sm)"]
        SpaCy --> HLG["Build Hierarchical Legal Graph\n(HLG)"]

        HLG -- Detect --> Cues["Legal Cues\n- Obligation: shall, must, agree, will\n- Conditions: if, unless, provided that"]
        HLG -- Extract --> Triplets["Extracted Triplets\n(Clause -> refers_to -> Entity)"]
    end

    %% EVALUATION (BOTTOM)
    subgraph Evaluation["Evaluation Phase"]
        GenSummary --> Metrics["Linguistic Metrics\n- ROUGE\n- BLEU\n- METEOR\n- chrF"]

        GenSummary --> LF["LegalFaith Metric"]
        Triplets --> LF

        LF --> FinalScore["Final Fidelity Score"]
    end

    %% Styling
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    class Input,FinalScore plain;
