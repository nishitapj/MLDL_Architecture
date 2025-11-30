# MLDL_Architecture
graph TD
    %% Core Inputs
    Input([Raw Legal Documents<br/>BillSum Corpus]) --> Norm[Text Normalization]

    %% Region 1: The Generative Model (Blue Modules in Fig 2)
    subgraph Generative_Path [Generative Pipeline]
        style Generative_Path fill:#e6f3ff,stroke:#333,stroke-width:2px
        Norm --> Tokenize[T5 Tokenizer<br/>Prefix: 'summarize:']
        Tokenize --> Dataset[BillSumDataset<br/>Max Input: 320<br/>Max Target: 128]
        Dataset --> Model[T5-Small Transformer<br/>Encoder-Decoder]
        
        %% Training Details
        Optimizer[AdamW Optimizer] -.-> Model
        Scheduler[Linear Scheduler<br/>with Warmup] -.-> Model
        Grad[Gradient Accumulation<br/>& Clipping] -.-> Model
        
        Model --> Beam[Beam Search Decoding<br/>Beam Width=4]
        Beam --> Summary[Generated Summary]
    end

    %% Region 2: The Structural Verification (Green Modules in Fig 2)
    subgraph Verification_Path [Structural Verification Pipeline]
        style Verification_Path fill:#e6ffec,stroke:#333,stroke-width:2px
        Norm --> SpaCy[SpaCy Processing<br/>en_core_web_sm]
        SpaCy --> HLG_Build[Build Hierarchical<br/>Legal Graph]
        
        HLG_Build -- Identify --> Cues[Legal Cues<br/>'shall', 'if', 'must']
        HLG_Build -- Extract --> Triplet[Fact Triplets<br/>Subject-Relation-Object]
    end

    %% Evaluation & Convergence
    subgraph Eval [Evaluation Phase]
        Summary --> Metrics[Linguistic Metrics<br/>ROUGE, BLEU, METEOR]
        Summary --> LegalFaith{LegalFaith Metric}
        Triplet --> LegalFaith
        LegalFaith --> Score([Final Fidelity Score])
    end

    %% Styling
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    class Input,Score plain;
