IMBALMED Inception Runner (mPower audio)
========================================

What happens, in order
----------------------
1) **Scan melSpec images**: read `data/melSpec/*.jpg` into a table with `healthCode, record_id, column_id, image_rel_path`.
2) **Load labels**: read `data/data_paired/csv/Demographics_Survey.csv`, map `professional-diagnosis` false->0 / true->1.
3) **Join images + labels**: inner-join on `healthCode` to keep only labeled images.
4) **Load patient splits**:
   - Train patients per subdb: `data/data_paired/10_fold_CV/processed_audio/audio_imbalmed_splits/subdb{k}_train_XX_YY.csv`.
   - Shared val/test patients: `.../common_val_test_all_subdbs.csv`.
5) **Expand to image-level**:
   - For each fold (0-9) and subset (train/val/test), duplicate patient rows across all their images.
   - Keep columns `healthCode, record_id, column_id, image_rel_path, label, fold, subset`.
   - Write one combined CSV per subdb to `data/data_paired/10_fold_CV/processed_audio/audio_imbalmed_combined/audio_imbalmed_subdb{k}_combined.csv`.
6) **Train with internal CV**:
   - `train_model(csv_file=..., fold=-1, config={"use_fold_column": True, ...})` loops over all folds present in the CSV, trains/validates/tests per fold, and writes a CV summary JSON to `outputs/audio_imbalmed/subdb{k}/`.

Table previews (first 5 rows)
-----------------------------
- Scanned melSpec index  
| healthCode | record_id | column_id | image_rel_path |
| --- | --- | --- | --- |
| 00081bd9-9abd-4003-b035-de6cc3e8c922 | f5f9fe7a-16ea-4729-ad36-4adf16d640d3 | audio_audio_m4a | data/melSpec/00081bd9-9abd-4003-b035-de6cc3e8c922_f5f9fe7a-16ea-4729-ad36-4adf16d640d3_audio_audio_m4a.jpg |
| 0160664a-f4af-4071-a4aa-2967f3ea0503 | 04de04ab-34b7-45ce-9cb5-59465eb51602 | audio_audio_m4a | data/melSpec/0160664a-f4af-4071-a4aa-2967f3ea0503_04de04ab-34b7-45ce-9cb5-59465eb51602_audio_audio_m4a.jpg |
| 0160664a-f4af-4071-a4aa-2967f3ea0503 | 07694dc8-0fe3-434c-a3a7-4d981d0a435b | audio_audio_m4a | data/melSpec/0160664a-f4af-4071-a4aa-2967f3ea0503_07694dc8-0fe3-434c-a3a7-4d981d0a435b_audio_audio_m4a.jpg |
| 0160664a-f4af-4071-a4aa-2967f3ea0503 | 0c466425-cd90-468d-ac9a-cf9d322a4729 | audio_audio_m4a | data/melSpec/0160664a-f4af-4071-a4aa-2967f3ea0503_0c466425-cd90-468d-ac9a-cf9d322a4729_audio_audio_m4a.jpg |
| 0160664a-f4af-4071-a4aa-2967f3ea0503 | 128091f2-7350-4f48-99fe-c99137e6c94d | audio_audio_m4a | data/melSpec/0160664a-f4af-4071-a4aa-2967f3ea0503_128091f2-7350-4f48-99fe-c99137e6c94d_audio_audio_m4a.jpg |

- Labels (Demographics_Survey)  
| healthCode | professional-diagnosis |
| --- | --- |
| 639e8a78-3631-4231-bda1-c911c1b169e5 | False |
| 9295f618-177c-4676-b6aa-dc8419fd37ec |  |
| 52fe366a-2a9f-4260-9fb1-0fbc637a6cf4 | False |
| 67bdd316-26fc-4fc7-8431-bf9f41a649dd | False |
| 45b4e2ca-8d15-4736-828c-829e3d4177f4 | False |

- Patient split train (example: subdb1)  
| fold_iteration | healthCode | subset |
| --- | --- | --- |
| 0 | 00081bd9-9abd-4003-b035-de6cc3e8c922 | train |
| 0 | 0160664a-f4af-4071-a4aa-2967f3ea0503 | train |
| 0 | 01aea090-d2cd-409e-8ce8-22887ada7af8 | train |
| 0 | 01b1ec31-0348-4148-a641-626d4391b4fb | train |
| 0 | 02c7f33f-0dd9-41e1-adee-1d6f107c87a2 | train |

- Shared val/test split (all subdbs)  
| fold_iteration | healthCode | subset |
| --- | --- | --- |
| 0 | 01fa5bda-f11e-47ef-94c6-37697ad26a86 | val |
| 0 | 030fe814-de84-4fb5-8df6-84dd6e7cc7dc | val |
| 0 | 05d4e71a-c792-4549-8ec7-6a7dc355a1bd | val |
| 0 | 09670b1f-23a3-46a5-8daf-f78f577b7697 | val |
| 0 | 13627002-073b-4366-8889-9561789c51be | val |

- Generated combined CSV (example: subdb1)  
| healthCode | health_code | record_id | column_id | image_rel_path | label | fold | subset |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 03d50e89-143a-4084-bc25-4b13077ff381 | 03d50e89-143a-4084-bc25-4b13077ff381 | 9fa3d979-15dd-42ee-8b9e-8b3c547abbcf | audio_audio_m4a | data/melSpec/03d50e89-143a-4084-bc25-4b13077ff381_9fa3d979-15dd-42ee-8b9e-8b3c547abbcf_audio_audio_m4a.jpg | 0 | 0 | test |
| 03d50e89-143a-4084-bc25-4b13077ff381 | 03d50e89-143a-4084-bc25-4b13077ff381 | 9e77ef98-d4a7-444e-85c5-a0d6d42ac370 | audio_audio_m4a | data/melSpec/03d50e89-143a-4084-bc25-4b13077ff381_9e77ef98-d4a7-444e-85c5-a0d6d42ac370_audio_audio_m4a.jpg | 0 | 0 | test |
| 03d50e89-143a-4084-bc25-4b13077ff381 | 03d50e89-143a-4084-bc25-4b13077ff381 | 23e0afc6-0aef-4d73-bc1d-c394d2ceca48 | audio_audio_m4a | data/melSpec/03d50e89-143a-4084-bc25-4b13077ff381_23e0afc6-0aef-4d73-bc1d-c394d2ceca48_audio_audio_m4a.jpg | 0 | 0 | test |
| 03d50e89-143a-4084-bc25-4b13077ff381 | 03d50e89-143a-4084-bc25-4b13077ff381 | 07a46c57-0cc8-4e2f-9291-6def3bd8f6a3 | audio_audio_m4a | data/melSpec/03d50e89-143a-4084-bc25-4b13077ff381_07a46c57-0cc8-4e2f-9291-6def3bd8f6a3_audio_audio_m4a.jpg | 0 | 0 | test |
| 03d50e89-143a-4084-bc25-4b13077ff381 | 03d50e89-143a-4084-bc25-4b13077ff381 | a7a08553-218e-4114-979c-655120886dc7 | audio_audio_m4a | data/melSpec/03d50e89-143a-4084-bc25-4b13077ff381_a7a08553-218e-4114-979c-655120886dc7_audio_audio_m4a.jpg | 0 | 0 | test |

How to run
----------
- Build CSVs + train all subdbs with internal CV:
  `python src_dp_team/model_audio/inception_imbalmed_runner.py --epochs 50 --batch-size 32 --overwrite`
- Limit to selected subdbs:
  `python src_dp_team/model_audio/inception_imbalmed_runner.py --subdbs 1 2 5 --overwrite`

Notes
-----
- `--overwrite` forces regeneration of combined CSVs (useful if folds were missing).
- `train_model` writes a CV summary per method: `outputs/audio_imbalmed/subdb{k}/{method}_cv_summary.json`.
