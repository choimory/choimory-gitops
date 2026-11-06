# 백엔드 API 명세서 규칙

- 모든 API에 대해 OpenAPI Json 방식으로 API 명세서를 작성한다
- 기존 API의 변경이 이루어졌을시 기존 OpenAPI 파일에도 반영한다 
- OpenAPI Json 파일은 프로젝트 내 도메인별로 구분한다
- 파일은 docs/api/{version}/{project-name}-openapi-{domain}-{version}.json에 작성한다
  - 예시: docs/api/{v1}/{choimory-dev-member-api}-openapi-{member}-{v1}.json
- description에는 설명을, example에는 예시 값을 작성한다
- 각각의 API에 대해 요청/응답 타입별로 예시 값을 모두 작성한다 (하나의 API에서 요청 타입이 A, B, C일때.. / 응답 코드가 200일때, 404일때, 500일때..)

## 예시

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "choimory-dev-member-api - member",
    "version": "v1",
    "description": "회원(member) 도메인 API 명세서입니다.\n\n- 모든 응답은 공통 응답 객체(`code`, `name`, `message`, `data`)로 내려갑니다.\n- 비즈니스 예외(중복 등)는 `CommonException`으로 처리되어 `name`에 오류 코드명이 담기고 `data`는 null입니다.\n- 요청 값 검증 실패는 400이며 `data`에 실패한 필드 목록이 담깁니다."
  },
  "servers": [
    {
      "url": "http://localhost:8080",
      "description": "로컬"
    }
  ],
  "tags": [
    {
      "name": "member",
      "description": "회원"
    }
  ],
  "paths": {
    "/v1/member/join": {
      "post": {
        "tags": [
          "member"
        ],
        "operationId": "joinMember",
        "summary": "회원가입",
        "description": "이메일, 비밀번호, 닉네임으로 회원가입합니다.\n\n- 가입한 회원은 즉시 `ACTIVATE` 상태가 되며 기본 권한 `MEMBER`가 부여됩니다.\n- 이메일과 닉네임은 중복될 수 없습니다. 탈퇴(soft delete)한 회원의 이메일과 닉네임도 hard delete 배치 전까지는 사용할 수 없습니다.\n- 비밀번호는 암호화되어 저장되며 응답에 포함되지 않습니다.\n- 약관 동의와 이메일 인증은 이후 추가됩니다.",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/MemberJoinRequest"
              },
              "examples": {
                "정상 요청": {
                  "summary": "모든 필드를 입력한 요청",
                  "value": {
                    "email": "choi@example.com",
                    "password": "Password1!",
                    "nickname": "choimory",
                    "introduce": "안녕하세요. 개발자 최모리입니다."
                  }
                },
                "자기소개 생략": {
                  "summary": "선택 값인 자기소개를 생략한 요청",
                  "value": {
                    "email": "choi2@example.com",
                    "password": "Password1!",
                    "nickname": "최모리",
                    "introduce": null
                  }
                },
                "한글 닉네임": {
                  "summary": "한글, 영문, 숫자를 섞은 닉네임",
                  "value": {
                    "email": "choi3@example.com",
                    "password": "Abcd1234@",
                    "nickname": "최모리Dev1",
                    "introduce": "백엔드 개발자"
                  }
                }
              }
            }
          }
        },
        "responses": {
          "201": {
            "description": "회원가입 성공",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/CommonResponseMemberJoinResponse"
                },
                "examples": {
                  "성공": {
                    "summary": "회원가입 성공",
                    "value": {
                      "code": 201,
                      "name": "CREATED",
                      "message": "Created",
                      "data": {
                        "id": "0199f3a2-7c1e-7b52-9d3a-4f6e8a1b2c3d",
                        "email": "choi@example.com",
                        "nickname": "choimory",
                        "status": "ACTIVATE"
                      }
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "요청 값 검증 실패. `data`에 실패한 필드 목록이 담깁니다. 한 필드가 여러 규칙을 위반하면 항목이 여러 개 담길 수 있습니다.",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/CommonResponseValidateResults"
                },
                "examples": {
                  "검증 실패": {
                    "summary": "이메일 형식, 비밀번호 구성, 닉네임 문자 규칙 위반",
                    "value": {
                      "code": 400,
                      "name": "BAD_REQUEST",
                      "message": "Bad Request",
                      "data": [
                        {
                          "field": "email",
                          "rejectValue": "a@b",
                          "message": "올바른 이메일 형식이 아닙니다."
                        },
                        {
                          "field": "password",
                          "rejectValue": "password1",
                          "message": "영문, 숫자, 특수문자를 모두 포함해야 합니다."
                        },
                        {
                          "field": "nickname",
                          "rejectValue": "a_b",
                          "message": "닉네임은 한글, 영문, 숫자만 사용해 3~15자여야 합니다."
                        }
                      ]
                    }
                  }
                }
              }
            }
          },
          "409": {
            "description": "이메일 또는 닉네임 중복. 탈퇴(soft delete)한 회원의 값도 중복으로 판정합니다. 사전 검증을 통과한 동시 요청이 저장 시점에 유니크 제약에 걸린 경우에도 같은 응답입니다.",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/CommonResponseError"
                },
                "examples": {
                  "이메일 중복": {
                    "summary": "이미 사용 중인 이메일",
                    "value": {
                      "code": 409,
                      "name": "DUPLICATE_EMAIL",
                      "message": "이미 사용 중인 이메일입니다.",
                      "data": null
                    }
                  },
                  "닉네임 중복": {
                    "summary": "이미 사용 중인 닉네임",
                    "value": {
                      "code": 409,
                      "name": "DUPLICATE_NICKNAME",
                      "message": "이미 사용 중인 닉네임입니다.",
                      "data": null
                    }
                  }
                }
              }
            }
          },
          "500": {
            "description": "서버 오류. 내부 정보 노출을 막기 위해 원인은 응답에 담지 않습니다. (예: 기본 권한 데이터 누락)",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/CommonResponseError"
                },
                "examples": {
                  "서버 오류": {
                    "summary": "처리되지 않은 서버 오류",
                    "value": {
                      "code": 500,
                      "name": "INTERNAL_SERVER_ERROR",
                      "message": "Internal Server Error",
                      "data": null
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "MemberJoinRequest": {
        "type": "object",
        "description": "회원가입 요청",
        "required": [
          "email",
          "password",
          "nickname"
        ],
        "properties": {
          "email": {
            "type": "string",
            "maxLength": 255,
            "format": "email",
            "pattern": "^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$",
            "description": "이메일. 도메인에 점이 있어야 합니다. (예: a@b 불가)",
            "example": "choi@example.com"
          },
          "password": {
            "type": "string",
            "minLength": 8,
            "maxLength": 64,
            "pattern": "^(?=.*[A-Za-z])(?=.*\\d)(?=.*[^A-Za-z\\d]).+$",
            "description": "비밀번호. 영문, 숫자, 특수문자를 모두 포함한 8~64자",
            "example": "Password1!"
          },
          "nickname": {
            "type": "string",
            "pattern": "^[가-힣A-Za-z0-9]{3,15}$",
            "description": "닉네임. 특수문자와 공백을 제외한 한글, 영문, 숫자 3~15자",
            "example": "choimory"
          },
          "introduce": {
            "type": [
              "string",
              "null"
            ],
            "maxLength": 500,
            "description": "자기소개 (선택, 500자 이하)",
            "example": "안녕하세요. 개발자 최모리입니다."
          }
        }
      },
      "MemberJoinResponse": {
        "type": "object",
        "description": "회원가입 응답. 비밀번호는 포함하지 않습니다.",
        "properties": {
          "id": {
            "type": "string",
            "description": "회원 ID (UUID v7)",
            "example": "0199f3a2-7c1e-7b52-9d3a-4f6e8a1b2c3d"
          },
          "email": {
            "type": "string",
            "description": "이메일",
            "example": "choi@example.com"
          },
          "nickname": {
            "type": "string",
            "description": "닉네임",
            "example": "choimory"
          },
          "status": {
            "$ref": "#/components/schemas/MemberStatus"
          }
        }
      },
      "MemberStatus": {
        "type": "string",
        "enum": [
          "REGISTER",
          "ACTIVATE",
          "INACTIVATE",
          "SUSPENDED"
        ],
        "description": "회원 상태. REGISTER: 가입(인증 대기), ACTIVATE: 활성, INACTIVATE: 비활성, SUSPENDED: 정지. 가입 직후에는 ACTIVATE입니다.",
        "example": "ACTIVATE"
      },
      "CommonValidateResult": {
        "type": "object",
        "description": "요청 값 검증 실패 정보",
        "properties": {
          "field": {
            "type": "string",
            "description": "검증에 실패한 필드명",
            "example": "email"
          },
          "rejectValue": {
            "description": "거부된 값",
            "example": "a@b"
          },
          "message": {
            "type": "string",
            "description": "검증 실패 메시지",
            "example": "올바른 이메일 형식이 아닙니다."
          }
        }
      },
      "CommonResponseMemberJoinResponse": {
        "type": "object",
        "description": "성공 응답 (data: 회원가입 결과)",
        "properties": {
          "code": {
            "type": "integer",
            "description": "응답 코드",
            "example": 201
          },
          "name": {
            "type": "string",
            "description": "응답 코드명",
            "example": "CREATED"
          },
          "message": {
            "type": "string",
            "description": "응답 메시지",
            "example": "Created"
          },
          "data": {
            "$ref": "#/components/schemas/MemberJoinResponse"
          }
        }
      },
      "CommonResponseValidateResults": {
        "type": "object",
        "description": "검증 실패 응답 (data: 실패한 필드 목록)",
        "properties": {
          "code": {
            "type": "integer",
            "description": "응답 코드",
            "example": 400
          },
          "name": {
            "type": "string",
            "description": "응답 코드명",
            "example": "BAD_REQUEST"
          },
          "message": {
            "type": "string",
            "description": "응답 메시지",
            "example": "Bad Request"
          },
          "data": {
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/CommonValidateResult"
            }
          }
        }
      },
      "CommonResponseError": {
        "type": "object",
        "description": "오류 응답 (data 없음)",
        "properties": {
          "code": {
            "type": "integer",
            "description": "응답 코드",
            "example": 409
          },
          "name": {
            "type": "string",
            "description": "오류 코드명",
            "example": "DUPLICATE_EMAIL"
          },
          "message": {
            "type": "string",
            "description": "오류 메시지",
            "example": "이미 사용 중인 이메일입니다."
          },
          "data": {
            "type": "null",
            "description": "데이터 없음",
            "example": null
          }
        }
      }
    }
  }
}

```