# Skills

Coleção de skills para o [Claude Code](https://claude.com/claude-code).

## Skills disponíveis

| Skill | O que faz |
|-------|-----------|
| [motion-reel](motion-reel/SKILL.md) | Cria vídeos de motion design (showreels, explainers, anúncios) renderizados 100% a partir de código, com HTML + Playwright + ffmpeg. |

## Como instalar

Copie a pasta da skill para a pasta de skills do Claude Code:

```bash
git clone https://github.com/Kstilho/claude-skill.git
cp -r claude-skill/motion-reel ~/.claude/skills/
```

Ou, para usar só em um projeto, copie para `.claude/skills/` dentro do projeto.

## Estrutura

```
claude-skill/
├── README.md
└── motion-reel/
    └── SKILL.md
```

Cada skill fica em sua própria pasta, com um `SKILL.md` contendo o cabeçalho (`name` e `description`) e as instruções.
