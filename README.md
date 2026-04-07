# Fitness App — Seed Data Package

## Contenu

| Fichier | Table(s) cible | Nb records |
|---------|---------------|-----------|
| `01_personas.json` | `personas` | 5 |
| `02_programs.json` | `programs` + `program_phases` | 5 programs / 13 phases |
| `03_persona_program_eligibility.json` | `persona_program_eligibility` | 25 (5×5 matrice) |
| `04_questionnaire_initial.json` | `questionnaire_questions` + `questionnaire_options` | 9 questions / 35 options |
| `05_feedback_poll.json` | `feedback_poll_questions` + `feedback_poll_options` | 3 questions / 16 options |
| `06_program_variants.json` | `program_variants` | 6 variants |
| `07_alternative_pitches.json` | `alternative_program_pitches` | 4 pitches |
| `08_exercises.json` | `exercises` | ~95 exercises |
| `schema.sql` | DDL complet | 15 tables |

## Tables référence (seed) vs runtime

### Seed (données statiques — injectées en Session 1)
- `personas`
- `programs`
- `program_phases`
- `persona_program_eligibility`
- `exercises`
- `questionnaire_questions`
- `questionnaire_options`
- `feedback_poll_questions`
- `feedback_poll_options`
- `program_variants`
- `alternative_program_pitches`

### Runtime (données utilisateur — créées en production)
- `users`
- `user_programs`
- `sessions`
- `session_logs`
- `feedback_responses`
- `subscriptions`

## Workout Generator — inputs/outputs

**Input** :
```python
{
  "persona_id": "persona_smb",
  "program_id": "program_muscle_building",
  "phase_id": "mbf_phase_2",
  "week_number": 3,
  "protocol": "upper_lower",
  "focus": "upper",
  "energy_level": 4,  # 1-5 slider
  "available_equipment": ["barbell", "dumbbells", "cables", "rack"]
}
```

**Output** :
```python
{
  "warmup_block": [...],   # 3-5 exercises spécifiques au focus du jour
  "main_block": [...],     # 4-6 exercises avec sets/reps/load/superset_with
  "core_block": [...],     # 2-3 exercises
  "finisher_block": [...]  # 1 finisher rotatif
}
```

## Order d'injection en Session 1

```bash
# 1. Créer le schema
psql $DATABASE_URL < schema.sql

# 2. Injecter dans cet ordre (FK dependencies)
python seed.py --file 02_programs.json      # programs + phases (avant personas FK)
python seed.py --file 01_personas.json      # personas (FK vers programs)
python seed.py --file 03_persona_program_eligibility.json
python seed.py --file 04_questionnaire_initial.json
python seed.py --file 05_feedback_poll.json
python seed.py --file 06_program_variants.json
python seed.py --file 07_alternative_pitches.json
python seed.py --file 08_exercises.json
```

## Schéma fonctionnel → mapping tables

```
[Questionnaire 8Q]  →  questionnaire_questions + questionnaire_options
         ↓
[5 Personae]        →  personas
         ↓
[Program Selection] →  persona_program_eligibility (rank: primary/secondary/tertiary/excluded)
         ↓
[Program + Phases]  →  programs + program_phases
         ↓
[Workout Generator] →  exercises (seed) → sessions (runtime)
         ↓
[Session Tracking]  →  session_logs (runtime)
         ↓
[Feedback Poll]     →  feedback_poll_questions + feedback_poll_options
         ↓
[Redirection]       →  program_variants + alternative_program_pitches → feedback_responses (runtime)
```
