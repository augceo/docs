# Глава 2.1: Примеры инструкций

## Пример 1: Планирование процесса

Этот пример показывает, как структурировать сложный, многоэтапный процесс, который включает в себя разные контексты выполнения (LLM, сервер, пользователь). Процесс предназначен для планирования встречи и требует определения участников, проверки их доступности, выбора временного слота и управления приглашениями. Каждое переключение контекста явно смоделировано, а зависимости данных между шагами четко определены через ссылки. Процесс показывает, как обрабатывать и немедленные операции LLM, и длительные задачи сервера в рамках единого, согласованного рабочего процесса.

**Вложенное выражение:**

```ts
// Send the approved invitation
SendInvitation_Activity(
  // Get user approval for the drafted invitation
  ConfirmInvitation_User(
    // Create invitation with participants and selected time slot
    DraftInvitation(
      // Extract participant info from user's request
      participants: IdentifyParticipants(prompt),
      // Find a time that works for both participants
      slot: FindCommonSlot(
        // Get calendar data for identified participants
        FetchAvailability_Activity(IdentifyParticipants(prompt)))
      )
    )
  )
)
```

Или альтернативно:

```typescript
// Extract participant info from user's request
const participants = IdentifyParticipants(prompt);

// Get calendar data for identified participants
const availabilityParams = FetchAvailability(participants);
const availabilityData = FetchAvailability_Activity(availabilityParams);

// Find a time that works for both participants
const commonSlot = FindCommonSlot(availabilityData);

// Create invitation with participants and selected time slot
const invitation = DraftInvitation(participants, commonSlot);

// Get user approval for the drafted invitation
const approval = ConfirmInvitation_User(invitation);

// Send the approved invitation
const result = SendInvitation_Activity(approval);
```

Упрощенное выражение в псевдокоде:

```ts
Process(
  // LLM определяет участников и готовит параметры запроса к календарю
  LLM_Context({
    participants: IdentifyParticipants(prompt),
    availabilityParams: FetchAvailability({ participants }),
  }),

  // Сервер получает актуальные данные календаря
  Server_Context({
    availabilityData: FetchAvailability_Activity({ availabilityParams }),
  }),

  // LLM анализирует доступность и создает приглашение
  LLM_Context({
    commonSlot: FindCommonSlot({ availabilityData }),
    invitation: DraftInvitation({ participants, commonSlot }),
  }),

  // Пользователь просматривает и одобряет приглашение
  User_Context({
    approval: ConfirmInvitation_User({ invitation }),
  }),

  // Сервер отправляет одобренное приглашение
  Server_Context({
    result: SendInvitation_Activity({ invitation, approval }),
  })
);
```

Этот пример демонстрирует, как сложный, многоэтапный процесс запускается с помощью простой, сфокусированной инструкции. У процесса много шагов (определение участников, получение доступности, поиск слотов, создание и подтверждение приглашений), но начальная инструкция фокусируется только на том, что необходимо для старта.

Важно отметить, что только первый контекст LLM (`LLM_Context` с `IdentifyParticipants` и `FetchAvailability`) служит точкой входа для всего процесса — это начальное взаимодействие с LLM становится запускающей логикой, которая стартует рабочий процесс. После этого оно становится частью Vibe процесса, который переносит состояние через весь его жизненный цикл, поддерживая контекст и зависимости данных между шагами. Это гарантирует, что каждое последующее действие (выполняемое сервером, LLM или пользователем) имеет доступ к необходимой информации с предыдущих шагов. Vessel лишь облегчает это взаимодействие, но сам не становится процессом.

### Схема процесса

Эта схема является частью Vibe `aug:processes/schedule-meeting:1`, который существует как прототип полного рабочего процесса планирования встречи со всеми его шагами и зависимостями данных. Далее в этом объяснении мы покажем, как Vessel может создать экземпляр этого процесса, завершив его первый шаг в работе с контекстом LLM.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Процесс: Планирование встречи",
  "description": "Надежный процесс для планирования встречи, включающий шаги LLM и сервера.",
  "type": "object",
  "properties": {
    "llmContext1": {
      "title": "Начальная настройка и подготовка параметров",
      "description": "LLM определяет участников и готовит параметры для операций сервера",
      "type": "object",
      "properties": {
        "identifyParticipants1": {
          "title": "Определение участников встречи",
          "type": "object",
          "properties": {
            "organizer": {
              "type": "string",
              "description": "Человек, инициирующий встречу."
            },
            "attendee": {
              "type": "string",
              "description": "Человек, которого приглашают."
            }
          },
          "required": ["organizer", "attendee"]
        },
        "fetchAvailability": {
          "title": "Подготовка параметров для запроса доступности",
          "references": ["identifyParticipants"],
          "type": "object",
          "properties": {
            "organizerId": {
              "type": "string",
              "description": "Системный идентификатор организатора, полученный из identifyParticipants.organizer."
            },
            "attendeeId": {
              "type": "string",
              "description": "Системный идентификатор участника, полученный из identifyParticipants.attendee."
            },
            "timeRange": {
              "type": "object",
              "properties": {
                "start": {
                  "type": "string",
                  "format": "date-time",
                  "description": "Начало временного диапазона для проверки доступности."
                },
                "end": {
                  "type": "string",
                  "format": "date-time",
                  "description": "Конец временного диапазона для проверки доступности."
                }
              },
              "required": ["start", "end"]
            }
          },
          "required": ["organizerId", "attendeeId", "timeRange"]
        }
      },
      "required": ["identifyParticipants", "fetchAvailability"]
    },
    "serverContext1": {
      "title": "Получение и обработка данных",
      "description": "Сервер получает и обрабатывает данные о доступности из календаря",
      "type": "object",
      "properties": {
        "FetchAvailability_Activity": {
          "title": "Результаты доступности в календаре",
          "references": ["llmContext1.fetchAvailability"],
          "type": "object",
          "properties": {
            "organizerSlots": {
              "type": "array",
              "items": { "type": "string", "format": "date-time" },
              "description": "Доступные временные слоты для организатора."
            },
            "attendeeSlots": {
              "type": "array",
              "items": { "type": "string", "format": "date-time" },
              "description": "Доступные временные слоты для участника."
            }
          },
          "required": ["organizerSlots", "attendeeSlots"]
        }
      },
      "required": ["FetchAvailability_Activity"]
    },
    "llmContext2": {
      "title": "Выбор времени встречи и черновик сообщения",
      "description": "LLM анализирует доступность и готовит приглашение на встречу",
      "type": "object",
      "properties": {
        "findCommonSlot": {
          "title": "Найти общий доступный слот",
          "references": [
            "llmContext1.identifyParticipants",
            "serverContext1.FetchAvailability_Activity"
          ],
          "type": "object",
          "properties": {
            "selectedSlot": {
              "type": "string",
              "format": "date-time",
              "description": "Выбранное время встречи, которое подходит обоим участникам."
            },
            "reasoning": {
              "type": "string",
              "description": "Объяснение, почему был выбран этот слот."
            }
          },
          "required": ["selectedSlot", "reasoning"]
        },
        "draftInvitation": {
          "title": "Черновик приглашения",
          "references": ["llmContext1.identifyParticipants", "findCommonSlot"],
          "type": "object",
          "properties": {
            "subject": { "type": "string" },
            "body": {
              "type": "string",
              "description": "Полный текст электронного письма с приглашением."
            }
          },
          "required": ["subject", "body"]
        }
      },
      "required": ["findCommonSlot", "draftInvitation"]
    },
    "userContext": {
      "title": "Точка принятия решения пользователем",
      "description": "Ожидает одобрения пользователем предложенных деталей встречи",
      "type": "object",
      "properties": {
        "confirmInvitation": {
          "title": "Получить подтверждение пользователя",
          "references": ["llmContext2.draftInvitation"],
          "type": "object",
          "properties": {
            "decision": {
              "type": "string",
              "enum": ["Approve", "Reject"],
              "description": "Решение пользователя по предложенной встрече."
            }
          },
          "required": ["decision"]
        }
      },
      "required": ["confirmInvitation"]
    },
    "serverContext2": {
      "title": "Окончательная отправка сообщения",
      "description": "Сервер отправляет одобренное приглашение на встречу",
      "type": "object",
      "properties": {
        "sendInvitation": {
          "title": "Отправить приглашение",
          "references": [
            "llmContext1.identifyParticipants",
            "llmContext2.draftInvitation",
            "userContext.confirmInvitation"
          ],
          "type": "object",
          "properties": {
            "messageId": { "type": "string" },
            "status": { "type": "string" }
          },
          "required": ["messageId", "status"]
        }
      },
      "required": ["sendInvitation"]
    }
  },
  "required": ["llmContext1", "serverContext1", "llmContext2", "userContext", "serverContext2"]
}
```

### Целенаправленный поток данных с `references`

Мета-свойство `references`, показанное в схеме выше, является ключом к управлению потоком данных и размером контекста. Движок рабочего процесса использует это свойство, чтобы добавлять в промпт для LLM только необходимые данные с предыдущих шагов для текущего шага.

Давайте рассмотрим конкретный пример. Представьте себе шаг `step5_draft_email`, которому нужно имя пользователя с предыдущего шага, `step4_find_user`. Схема будет выглядеть так:

```json
"step5_draft_email": {
    "description": "Составить персонализированное письмо пользователю, найденному на предыдущем шаге.",
    "references": [ "step4_find_user.output" ],
    "type": "object",
    "properties": {
        "recipientName": {
            "description": "Используйте поле 'userName' из объекта 'step4_find_user.output', предоставленного в контексте промпта.",
            "type": "string"
        },
        "emailBody": { "type": "string" }
    }
}
```

Движок рабочего процесса обрабатывает это так:

1.  Он видит `references: [ "step4_find_user.output" ]`.
2.  Он извлекает весь выходной объект `{ "userId": "...", "userName": "..." }` из своего состояния.
3.  Он создает минимальный контекст и добавляет его в промпт для LLM.

<pre>
--- Контекст с предыдущих шагов ---
{
    "step4_find_user": {
        "output": {
            "userId": "u-12345",
            "userName": "Jane Doe"
        }
    }
}
--- Конец контекста ---

Пожалуйста, сгенерируйте JSON для шага `step5_draft_email`, используя предоставленную выше информацию.
</pre>

4.  Он отправляет схему для `step5_draft_email` в LLM, но **без свойства `references`**. Затем LLM использует предоставленный контекст, чтобы следовать инструкциям в полях `description`. Это обеспечивает чистый и мощный механизм для передачи данных от действий на стороне сервера обратно в процесс рассуждений LLM.

Это гарантирует, что LLM получает только необходимую информацию, предотвращая перегрузку контекста и сохраняя эффективность взаимодействия. Большая схема `Процесс: Планирование встречи` широко использует этот шаблон для передачи информации между контекстами LLM, сервера и пользователя.

## Модульная схема

Вместо того чтобы показывать LLM всю сложную схему процесса, мы создаем инструмент специально для _запуска_ процесса. Этот инструмент создается путем взятия первого контекста LLM из Примера 1 и его улучшения дополнительными когнитивными шагами через композицию Инструкций.

Давайте проследим, как работает эта композиция, начиная с исходной структуры процесса.

### Использование инструкций для когнитивного улучшения

Инструкции могут компоноваться друг с другом, потому что их схемы сливаются подобно объектам JavaScript, а JSON Schema сохраняет порядок свойств. Это позволяет создавать макро-подобные Инструкции, которые оборачивают другие Инструкции, добавляя при этом дополнительные шаги:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Introspection_Mixin",
  "description": "Инструкция, которая улучшает любую схему, добавляя анализ перед действием и рефлексию после действия через естественную композицию",
  "type": "object",
  "properties": {
    "_considerations": {
      "type": "string",
      "description": "Перед выполнением действия проанализируйте запрос и выделите ключевые моменты."
    },
    "default": {
      "type": "object",
      "description": "Это заполнитель, который будет заменен схемой цели во время выполнения."
    },
    "_feedback": {
      "type": "string",
      "description": "После завершения действия предоставьте обратную связь о принятых решениях."
    }
  },
  "required": ["_considerations", "default", "_feedback"]
}
```

Когда эта Инструкция компонуется с другой, свойство `default` действует как слот, который заменяется свойствами другой схемы. Поскольку порядок свойств сохраняется, мы сначала получаем `_considerations`, затем все свойства из скомпонованной схемы, а затем `_feedback` — создавая макро-подобную обертку, которая добавляет шаги «мышления» вокруг основной задачи.

### Компоновка инструмента

Вот как мы превращаем первый контекст LLM из Примера 1 в наш инструмент:

Исходный фрагмент первого контекста LLM из процесса выглядит так:

```ts
const llmContext1 = FetchAvailability(IdentifyParticipants(prompt));
```

Vessel оборачивает его с запуском процесса и интроспекцией:

```ts
StartProcess({
  name: 'ScheduleMeeting',
  llmContext1: Introspect_Mixin(FetchAvailability(IdentifyParticipants(prompt))),
});
```

Introspect_Mixin — это миксин-инструкция, которая принимает именованные свойства:

```ts
StartProcess({
  name: 'ScheduleMeeting',
  llmContext1: Introspect_Mixin({
    default: {
      identifyParticipants: IdentifyParticipants(prompt),
      fetchAvailability: FetchAvailability({ participants }),
    },
  }),
});
```

Это приводит к композиции JSON-схем, где инструкции компонуются вместе. Обратите внимание, как мы используем поле `description` для указания, как передавать аргументы между шагами. Например, описание `fetchAvailability` указывает, что оно должно повторно использовать email-адреса участников с шага идентификации. Этот подход обеспечивает ясный поток данных между компонентами инструкции, не требуя явного объявления параметров.

```ts
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Запустить процесс планирования",
  "description": "Запускает рабочий процесс планирования встречи с определением участников и проверкой доступности",
  "type": "object",
  "allOf": [
    { "$ref": "aug:/vibes/macros/StartProcess:1" },
    {
      "properties": {
        "name": { "const": "ScheduleMeeting" },
        "llmContext1": {
          "type": "object",
          "allOf": [
            { "$ref": "aug:/vibes/macros/Introspect_Mixin" },
            { "$ref": "aug:/vibes/macros/Loop_Mixin", "description": "Цикл не более 3 раз" },
            {
              "properties": {
                "default": {
                  "type": "object",
                  "properties": {
                    "identifyParticipants": {
                      "allOf": [
                        { "$ref": "aug:instructions/identifyParticipants" },
                        { "description": "Извлекает и идентифицирует информацию об участниках из промпта пользователя, определяя, кто должен быть включен во встречу" }
                      ]
                    },
                    "fetchAvailability": {
                      "allOf": [
                        { "$ref": "aug:processes/schedule-meeting:1#fetchAvailability" },
                        { "description": "Получает данные о доступности из календаря для определенных участников, повторно используя их email-адреса с шага идентификации для проверки расписаний: llm_context_1.identifyParticipant" }
                      ]
                    }
                  }
                }
              }
            }
          ]
        }
      }
    }
  ]
}
```

Вот как это выглядит, когда миксин раскрыт, и LLM выполняет свою работу

> Промпт: Ассистент, запланируй встречу между Алисой и Бобом!

```ts
StartProcess({
  name: 'ScheduleMeeting',
  llmContext1: {
    _considerations: "Нужно проверить доступность обоих участников в рабочее время",
    identifyParticipants: {
      organizer: 'alice@example.com',
      attendee: 'bob@example.com',
    },
    fetchAvailability: {
      organizerEmail: 'alice@example.com', // Повторно используется из identifyParticipants
      attendeeEmail: 'bob@example.com', // Повторно используется из identifyParticipants
      timeRange: {
        start: '2024-03-20T09:00:00Z',
        end: '2024-03-20T17:00:00Z',
      },
    },
    _feedback:
      'Подготовлено к проверке доступности для Алисы и Боба в течение их рабочего дня. После завершения процесс продолжится поиском общих слотов.',
  },
});
```

Это показывает, как структура схемы направляет LLM:

1. Продумать важные моменты перед действием (`_considerations`)
2. Предоставить полные, хорошо структурированные данные для начальных операций
3. Повторно использовать email-адреса участников напрямую с шага идентификации
4. Сделать выводы как о немедленном шаге, так и о продолжении процесса (`_feedback`)
5. Явно запустить процесс `ScheduleMeeting` на сервере и продолжить его там

### Скомпонованная схема инструмента

Это окончательная схема, представленная LLM, со всеми разрешенными ссылками. Она объединяет собственные корректировки Vessel с определениями инструментов (например, инструкции о том, как передавать вывод одного шага на вход другого)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Инструментарий Vessel: Планировщик встреч",
  "description": "Набор инструментов, доступных Vessel для планирования встреч.",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "const": "ScheduleMeeting",
      "description": "Определяет запускаемый процесс"
    },
    "llmContext1": {
      "type": "object",
      "properties": {
        "_considerations": {
          "type": "string",
          "description": "Перед выполнением действия проанализируйте запрос и выделите ключевые моменты."
        },
        "identifyParticipants": {
          "title": "Определение участников встречи",
          "type": "object",
          "description": "Определяет участников встречи из ввода пользователя\nИзвлекает и идентифицирует информацию об участниках из промпта пользователя, определяя, кто должен быть включен во встречу",
          "properties": {
            "organizer": {
              "type": "string",
              "format": "email",
              "description": "Email человека, инициирующего встречу."
            },
            "attendee": {
              "type": "string",
              "format": "email",
              "description": "Email человека, которого приглашают."
            }
          },
          "required": ["organizer", "attendee"]
        },
        "fetchAvailability": {
          "title": "Получить доступность участников",
          "type": "object",
          "description": "Получает доступность в календаре для указанных участников\nПолучает данные о доступности из календаря для определенных участников, повторно используя их email-адреса с шага идентификации для проверки расписаний",
          "properties": {
            "organizerEmail": {
              "type": "string",
              "format": "email",
              "description": "Email организатора"
            },
            "attendeeEmail": {
              "type": "string",
              "format": "email",
              "description": "Email участника."
            },
            "timeRange": {
              "type": "object",
              "properties": {
                "start": {
                  "type": "string",
                  "format": "date-time",
                  "description": "Начало временного диапазона для проверки доступности."
                },
                "end": {
                  "type": "string",
                  "format": "date-time",
                  "description": "Конец временного диапазона для проверки доступности."
                }
              },
              "required": ["start", "end"]
            }
          },
          "required": ["organizerEmail", "attendeeEmail", "timeRange"]
        },
        "_feedback": {
          "type": "string",
          "description": "После завершения действия предоставьте обратную связь о принятых решениях."
        }
      },
      "required": ["_considerations", "identifyParticipants", "fetchAvailability", "_feedback"]
    }
  },
  "required": ["name", "llmContext1"]
}
```