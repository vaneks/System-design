- [Текстовая секция к 1 модулю](Description_1.md)
- [Context диаграмма](Context.puml)
- [C4 диаграмма](BankPlatform_C2.puml)

---

- [Текстовая секция к 2 модулю](Description_2.md)
- [C4 диаграмма](BankPlatform_Outbox_C2.puml)
- [C4 диаграмма Transaction Service](TransactionService.puml)
- [C4 диаграмма Wallet Service](WalletService.puml)
- [C4 диаграмма Query Service](QueryService.puml)
- [ERD диаграмма](Erd.puml)
- [Sequence диаграмма - сценарий "успешный платеж"](SequenceSuccess.puml)
- [Sequence диаграмма - сценарий "неуспешный платеж"](SequenceFail.puml)

---

- [Текстовая секция к 3 модулю](Description_3.md)
- [C4 диаграмма Cache](BankPlatform_Cache_C2.puml)
- [Sequence-диаграмма сценария: "клиент открывает историю платежей"](SequenceCache.puml)
- [C4 диаграмма Основные топики](Dql_topics_C2.puml)
- [Sequence-диаграмма сценария: "сообщение не удалось обработать несколько раз -> попадание в DLQ -> дальнейшая ручная/отложенная обработка"](Sequence_Topic_to_dql_C2.puml)
- [Sequence диаграмма - сценарий "провайдер тормозит/падает"](SequenceProvider.puml)
- [C4 диаграмма Observability](BankPlatform_Observability_C2.puml)