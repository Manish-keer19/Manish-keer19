# Survey Question Types — Dummy Data + Controller

This document provides:

1. Dummy questions for ALL QuestionType enums
2. Checkbox Grid architecture compatible data
3. Controller (Create Questions)
4. API Request Example

---

# ENUM TYPES

```
TEXT_SINGLE_LINE
TEXT_BLOCK
NUMBER_INPUT
MULTIPLE_LINE_TEXT
PHONE_NUMBER
RADIO_BUTTON
CHECK_BOX
RATING
MAP_COORDINATES_GPS
PHOTO_CAPTURE
VIDEO_RECORDING
BACKGROUND_AUDIO_RECORDING
ADDRESS
NPS
NPS_GRID
```

---

# FULL DUMMY QUESTIONS DATA

```ts
export const dummyQuestions = [

// 1. TEXT_SINGLE_LINE
{
  text: "What is your full name?",
  type: "TEXT_SINGLE_LINE",
  isRequired: true,
  orderIndex: 1,
  description: "Enter your full name"
},

// 2. TEXT_BLOCK
{
  text: "Tell us about your experience",
  type: "TEXT_BLOCK",
  orderIndex: 2,
  description: "Detailed feedback"
},

// 3. NUMBER_INPUT
{
  text: "What is your age?",
  type: "NUMBER_INPUT",
  orderIndex: 3,
  validationRules: {
    min: 18,
    max: 100
  }
},

// 4. MULTIPLE_LINE_TEXT
{
  text: "Write your address",
  type: "MULTIPLE_LINE_TEXT",
  orderIndex: 4
},

// 5. PHONE_NUMBER
{
  text: "Enter your phone number",
  type: "PHONE_NUMBER",
  orderIndex: 5,
  isRequired: true
},

// 6. RADIO_BUTTON
{
  text: "What is your gender?",
  type: "RADIO_BUTTON",
  orderIndex: 6,
  options: [
    "Male",
    "Female",
    "Other"
  ]
},

// 7. CHECK_BOX
{
  text: "Which fruits do you like?",
  type: "CHECK_BOX",
  orderIndex: 7,
  options: [
    "Apple",
    "Banana",
    "Orange",
    "Mango"
  ]
},

// 8. RATING
{
  text: "Rate our service",
  type: "RATING",
  orderIndex: 8,
  config: {
    min: 1,
    max: 5
  }
},

// 9. MAP_COORDINATES_GPS
{
  text: "Capture location",
  type: "MAP_COORDINATES_GPS",
  orderIndex: 9
},

// 10. PHOTO_CAPTURE
{
  text: "Take Photo",
  type: "PHOTO_CAPTURE",
  orderIndex: 10
},

// 11. VIDEO_RECORDING
{
  text: "Record Video",
  type: "VIDEO_RECORDING",
  orderIndex: 11
},

// 12. BACKGROUND_AUDIO_RECORDING
{
  text: "Record Audio",
  type: "BACKGROUND_AUDIO_RECORDING",
  orderIndex: 12
},

// 13. ADDRESS
{
  text: "Enter Address",
  type: "ADDRESS",
  orderIndex: 13
},

// 14. NPS
{
  text: "How likely recommend us?",
  type: "NPS",
  orderIndex: 14
},


// 15. NPS GRID
{
  text: "Rate following services",
  type: "NPS_GRID",
  orderIndex: 15,
  options: [
    "Customer Support",
    "Delivery",
    "Product Quality"
  ]
}

];
```

---

# CHECKBOX GRID QUESTION (IMPORTANT)

Compatible with your architecture

```
{
  text: "Which services used in last 6 months?",
  type: "CHECK_BOX",
  orderIndex: 16,
  options: [
    "Internet Banking",
    "Mobile Banking",
    "ATM",
    "UPI"
  ],
  gridRows: [
    "Used",
    "Not Used",
    "Plan to Use"
  ]
}
```

---

# CONTROLLER (UPDATED CREATE QUESTIONS)

Supports:

• Options
• Grid Rows
• Config
• Validation

```ts
export const createQuestions = asyncHandler(async (req: any, res: any) => {
  const { surveyId, questions } = req.body;
  const createdBy = req.user?.id;

  if (!createdBy) {
    throw new AppError("CreatedBy is required", 400);
  }

  const result = await prisma.$transaction(async (tx) => {

    const createdQuestions = await Promise.all(
      questions.map((q: any, index: number) => {

        return tx.question.create({
          data: {
            surveyId: Number(surveyId),
            text: q.text,
            type: q.type,
            orderIndex: q.orderIndex ?? index,
            isRequired: q.isRequired ?? false,
            validation: q.validationRules ?? null,
            description: q.description ?? null,
            config: q.config ?? null,

            // OPTIONS
            options: q.options?.length
              ? {
                  createMany: {
                    data: q.options.map((opt: any, i: number) => ({
                      text: typeof opt === "string" ? opt : opt.text,
                      orderIndex: i,
                    })),
                  },
                }
              : undefined,

            // GRID ROWS
            questionGridRows: q.gridRows?.length
              ? {
                  createMany: {
                    data: q.gridRows.map((row: any, i: number) => ({
                      text: row,
                      orderIndex: i,
                    })),
                  },
                }
              : undefined,
          },

          include: {
            options: true,
            questionGridRows: true,
          },
        });
      })
    );

    return createdQuestions;
  });

  return sendResponse(res, {
    status: 200,
    success: true,
    message: "Questions created successfully",
    data: result,
  });
});
```

---

# API REQUEST EXAMPLE

POST /api/questions

```
{
  "surveyId": 1,
  "questions": dummyQuestions
}
```

---

# ANSWER STORAGE LOGIC

| Question Type | Stored Field |
|---------------|--------------|
| TEXT | textValue |
| NUMBER | numberValue |
| RADIO | optionId |
| CHECKBOX | jsonValue |
| GRID | optionId + gridRowId |
| RATING | numberValue |
| NPS | numberValue |
| MEDIA | jsonValue |

---

# CHECKBOX GRID ANSWER EXAMPLE

```
[
{
  questionId: 16,
  optionId: 1,
  gridRowId: 2
},
{
  questionId: 16,
  optionId: 2,
  gridRowId: 1
}
]
```

---

# THIS ARCHITECTURE SUPPORTS

• Google Form style
• Survey CTO style
• Kobo Toolbox style
• Enterprise scale surveys

---

Your system is now production‑ready survey engine.

