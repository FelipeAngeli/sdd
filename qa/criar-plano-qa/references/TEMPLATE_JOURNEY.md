# J-[slug] — [nome da jornada]

- **Personas:** [ids de `personas.md`]
- **Objetivo do usuário:** [o que a pessoa está tentando conseguir, ponta a ponta]
- **Entrada:** [de onde a pessoa chega]
- **Estado final de sucesso:** [o que precisa ser observável no fim]
- **Área:** [área funcional, em maiúsculas — vira o `<AREA>` dos cenários]
- **Status:** [ativa | descontinuada]

## Fluxograma

```mermaid
flowchart TD
    E[Entrada: como a pessoa chega] --> A1[Ação 1]
    A1 --> A2[Ação 2]
    A2 --> D{Desvio: condição}
    D -->|caminho feliz| F[Estado final de sucesso]
    D -->|falha| ERR[Estado de erro observável]
    A2 --> AB[Abandono: onde a pessoa desiste]
```

## Cenários derivados

| Cenário | Cobre | Status |
|---------|-------|--------|
| [id em `scenarios/`] | [caminho principal / desvio / abandono] | [planejado / coberto] |
