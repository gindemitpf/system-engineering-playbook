
---

# Интерфейсы ICD-lite
<iframe src="https://gindemitpf.github.io/system-engineering-playbook/swagger/swagger.html" width="100%" height="1500px" style="border:0;" allowfullscreen="allowfullscreen"></iframe>
---

## 5.1. POST /api/v1/resumes

| Параметр | Описание |
| --- | --- |
| Назначение | Загрузка резюме для последующего анализа |
| Метод и путь | `POST /api/v1/resumes` |
| Заголовки | `Content-Type: multipart/form-data`, `X-Candidate-ID` |
| Ответ 202 | `resumeId`, `status: pending`, `message: "Resume accepted for processing"` |
| Ошибки | `400 INVALID_FILE_FORMAT`, `413 FILE_TOO_LARGE`, `500 INTERNAL_ERROR` |
| Идемпотентность | Да (по хэшу файла) |

## 5.2. GET /api/v1/resumes/{resumeId}/analysis

| Параметр | Описание |
| --- | --- |
| Назначение | Получение результатов интеллектуального анализа резюме |
| Метод и путь | `GET /api/v1/resumes/{resumeId}/analysis` |
| Ответ 200 | `status`, `extractedSkills: []`, `experienceYears`, `summary` |
| Ошибки | `404 RESUME_NOT_FOUND`, `422 PROCESSING_FAILED` |
| Идемпотентность | Да |

## 5.3. POST /api/v1/applications

| Параметр | Описание |
| --- | --- |
| Назначение | Привязка резюме к вакансии (создание отклика) |
| Метод и путь | `POST /api/v1/applications` |
| Тело запроса | `resumeId`, `vacancyId` |
| Ответ 201 | `applicationId`, `matchScore`, `status` |
| Ошибки | `400 INVALID_REQUEST`, `404 VACANCY_NOT_FOUND`, `409 ALREADY_APPLIED` |
| Идемпотентность | Да |

## 5.4. POST /api/v1/applications/{applicationId}/interview-questions

| Параметр | Описание |
| --- | --- |
| Назначение | Генерация уточняющих вопросов для кандидата через LLM |
| Метод и путь | `POST /api/v1/applications/{applicationId}/interview-questions` |
| Авторизация | Bearer JWT (HR) |
| Ответ 200 | `applicationId`, `questions: [string]`, `generatedAt` |
| Ошибки | `404 APPLICATION_NOT_FOUND`, `503 AI_SERVICE_UNAVAILABLE` |
| Идемпотентность | Нет |

## 5.5. GET /api/v1/vacancies/{vacancyId}/candidates

| Параметр | Описание |
| --- | --- |
| Назначение | Получение списка кандидатов, ранжированных по релевантности |
| Метод и путь | `GET /api/v1/vacancies/{vacancyId}/candidates` |
| Ответ 200 | Массив кандидатов: `applicationId`, `fullName`, `matchScore`, `status` |
| Ошибки | `404 VACANCY_NOT_FOUND` |

## 5.6. Событие ResumeUploaded

| Поле | Тип | Описание |
| --- | --- | --- |
| eventId | UUID | Уникальный ID события |
| eventType | string | `ResumeUploaded` |
| resumeId | UUID | ID файла в хранилище |
| candidateId | UUID | ID владельца резюме |

## 5.7. Событие ResumeAnalyzed

| Поле | Тип | Описание |
| --- | --- | --- |
| eventId | UUID | Уникальный ID события |
| eventType | string | `ResumeAnalyzed` |
| resumeId | UUID | ID резюме |
| status | string | `success` или `failed` |
| skillsCount | int | Количество найденных навыков |

## 5.8. Событие MatchScoreCalculated

| Поле | Тип | Описание |
| --- | --- | --- |
| eventId | UUID | Уникальный ID события |
| eventType | string | `MatchScoreCalculated` |
| applicationId | UUID | ID отклика |
| score | float | Итоговый рейтинг (0.0 - 1.0) |

**Ошибки обработки событий:** при сбоях в `AI Analysis Service` или `Recommendation Service`, события остаются в Message Broker (схема DLQ для переповторов).