# MLDL_Architecture
```mermaid
graph TD
    %% ================================
    %% INPUT STAGE
    %% ================================
    Input([Raw Legal Document<br/>BillSum: US/Cali Bills]) --> Pre[Text Preprocessing<br/>• Whitespace Normalization<br/>• Lowercasing (optional)]

    %% ================================
    %% TRAINING PATH (LEFT)
    %% ================================
    subgraph Training_Path [Model Training Path]
        style Training_Path fill:#e6f3ff,stroke:#333,stroke-width:2px

        Pre --> T5Token[T5 Tokenizer<br/>Prefix: 'summarize: ']
        T5Token --> Dataset[Custom BillSum Dataset<br/>Max Input=320<br/>Max Target=128]

        Dataset --> T5Model[T5-small Seq2Seq Model<br/>Encoder–Decoder]

        Optim[AdamW Optimizer] -.-> T5Model
        Sched[Linear Scheduler<br/>with Warmup + Decay] -.-> T5Model
        GradAcc[Gradient Accumulation<br/>(Eff. BS=8) + Clipping] -.-> T5Model

        T5Model --> Decode[Beam Search Decoding<br/>Beam Width=4]
        Decode --> GenSummary[Generated Summary<br/>(Abstractive)]
    end

    %% ================================
    %% TESTING / STRUCTURAL PATH (RIGHT)
    %% ================================
    subgraph Test_Path [Testing / Structural Verification Path]
        style Test_Path fill:#e6ffec,stroke:#333,stroke-width:2px

        Pre --> SpaCy[SpaCy Linguistic Processing<br/>en_core_web_sm]
        SpaCy --> HLG[Build Hierarchical Legal Graph (HLG)]

        HLG -- Detect --> Cues[Legal Cues<br/>• Obligation: shall, must, agree, will<br/>• Conditions: if, unless, provided that]
        HLG -- Extract --> Triplets[(Extracted Triplets<br/>(Clause → refers_to → Entity))]
    end

    %% ================================
    %% EVALUATION (BOTTOM)
    %% ================================
    subgraph Evaluation [Evaluation Phase]
        GenSummary --> Metrics[Linguistic Metrics<br/>• ROUGE • BLEU • METEOR • chrF]

        GenSummary --> LF[LegalFaith Metric]
        Triplets --> LF

        LF --> FinalScore([Final Fidelity Score])
    end

    %% ================================
    %% STYLING
    %% ================================
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    class Input,FinalScore plain;
