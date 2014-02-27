wisdio-api
==========

#WISDIO API draft

Version: 0.02
Creation date: 2011-09-13 15:22 CEST
Modification date: 2011-09-20 16:41 CEST

#Copyright Notice

Copyright (C) Wisdio S.A. (2014).  All Rights Reserved.

#Description

##Base URL

Base URL: http://api.wisdio.com/1 lub https://sapi.wisdio.com/1

##HTTP Method

All API method accept POST or GET if otherwise unspecified

##Authorization

  * Authorization: HTTP Basic see http://en.wikipedia.org/wiki/Basic_access_authentication
  * Username: email or login
  * Password: user password

Each request requires authorization.

##Standard responses

At least the following responses should be handled by client application.

  * 200 - OK: proceed with results
  * 400 - Invalid request: this should occur mainly because of bug in client application
  * 401 - Unauthorized: wrong user credentials
  * 403 - Access denied: e.g. wrong user agent
  * 404 - Not found: looked up resource is not found
  * 500 - Internal error: this should occur mainly because of bug in server application
  * 501 - Not implemented: some of descriped here functionality wasn't yet implemented
  * 502 - Bad gateway: this should occur mainly because of major server disaster
  * 503 - Service unavailable: this should occur mainly because of interdependent services disaster

##1. Register

Allows to register extended application method or user.

Method: *POST*

    [Base URL]/register/[app]|user/[method|lang]?[parameters]

###1b. User Registration

  * `lang`: language name (pl, en) for default profile and error messages
  * `parameters`:
    * `agreed`: "1" or ""
    * `fname`: Registering user first name
    * `lname`: Registering user last name
    * `email`: Registering user email address
    * `password`: Registering user password

example:

    [Base URL]/register/user/en?agreed=1&fname=John&lname=Doe&email=john%40example.com&password=12345

###Responses:

Response JSON:

    {'status': 'ok', 'uid': user_id}

  * `user_id`: String

the new user has been registered successfully

Response JSON:

    {'status': 'error', 'errors': errors, 'input': input_params}

  * `input_params`:

    {'agreed': agreed, 'fname': fname, 'lname': lname, 'email': email, 'password': password}

  * `agreed`: Boolean
  * `fname`: String
  * `lname`: String
  * `email`: String
  * `password`: String

  * `errors`:

    {['agreed': err_str][, 'fname': err_str][, 'lname': err_str][, 'email': err_str][, 'password': err_str]}

  * `err_str`: String

there where errors in some of the input fields while trying to register new user



###1b. Apple Push Notification

  * `app`: apple
  * `method`: push
  * `parameters`:
    * `token`: unique push identifier

example:

    [Base URL]/register/apple/push?token=0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

###Responses:

Response JSON:

    {'status': 'ok', 'uid': user_id, 'token': unique_push_identifier}

  * `user_id`: String
  * `unique_push_identifier`: String

the new identifier has been registered successfully

##2. Ask question

Allows to ask questions in the name of authorized user

Method: *POST*

    [Base URL]/ask?[parameters]

  * `parameters`:
    * `question`: question content string
    * `description` (optional): question description string
    * `topic` (optional/multiple): topic
    * `lang` (optional): override question language
    * `anonymous`=1 (optional): ask anonymously

If at least one topic is not specified the response is a list of proposed topics,
and detected language. 
If lang is not specified the response may contain a request for language confirmation.

###Responses:

Response JSON:

    {'status': 'ok', 'question': original_question_content, 'qid': question_identifier, 'url': shortened_url_link, 'cdate': creation_date}

  * `question_identifier`: String
  * `shortened_url_link`: String
  * `creation_date`: String (iso format, eg.: '2011-09-06T14:31:38.858121Z')

The question was successfully asked. This can only occur if at least one
topic was specified and language was successfully detected
(or lang parameter was specified).

Response JSON:

    {'status': 'confirm', 'question': original_question_content, 'langs': lang_probability_list, 'topics': topic_probability_list}

  * `original_question_content`: String
  * `topic_probability_list`: Array [[String, Number]]
  * `lang_probability_list`: Array [[String, Number]]

No tags or lang were specified and language was difficult to guess

Response JSON: 

    {'status': 'conflict', 'question': original_question_content, 'qid': question_identifier, 'url': shortened_url_link, 'cdate': creation_date}

  * `original_question_content`: String
  * `question_identifier`: String
  * `shortened_url_link`: String
  * `creation_date`: String (iso format, eg.: '2011-09-06T14:31:38.858121Z')

Someone asked the same question already (in future it will enable question following)

##3. Answer question

Allows to answer questions in the name of authorized user

Method: *POST*

    [Base URL]/addanswer/[qid]?[parameters]

  * `qid`: question identifier
  * `parameters`:
    * `body`: answer body (text/html)
    * `lang` (optional): message translation language (defaults to question language)

###Responses:

Response JSON:

    {'status': 'ok', 'body': sanitized_answer_body, 'aid': answer_identifier, 'qid': question_identifier, 'url': shortened_url_answer_link, 'cdate': creation_date}

  * `sanitized_answer_body`: String
  * `answer_identifier`: String
  * `question_identifier`: String
  * `shortened_url_answer_link`: String
  * `creation_date`: String (iso format, eg.: '2011-09-06T14:31:38.858121Z')

The question was successfully answered.

Response JSON:

    {'status': 'error', 'body': original_answer_body, 'qid': question_identifier, 'description': error_description}

  * `original_answer_body`: String
  * `question_identifier`: String
  * `error_description`: String

There was an error trying to answering question.

##4. Get question

Get question data.

    [Base URL]/question/[question_identifier]

  * `question_identifier`: retrieved qid

###Responses:

Response JSON:

    {'qid': question_identifier, 'lang': question_language, 'question': question_content, 'description': question_description, 'topics': topic_list, 'answers': answer_list, 'comments': comment_list, 'uid': author_id, 'cdate': creation_date}

  * `question_identifier`: String
  * `question_language`: String
  * `question_content`: String
  * `question_description`: String
  * `topic_list`: Array [String]
  * `answer_list`: Array [answer_description]
  * `comment_list`: Array [comment_description]
  * `author_id`: String or null (on anonymous questions)
  * `creation_date`: String (iso format)

  `answer_description`:

    {'aid': answer_identifier, 'lang': answer_language, 'answer': answer_body, 'score': answer_score, 'uid': author_id, 'comments': comment_list, 'cdate': creation_date}

  * `answer_identifier`: String
  * `answer_language`: String
  * `answer_body`: String
  * `answer_score`: Number
  * `author_id`: String
  * `comment_list`: Array [comment_description]
  * `creation_date`: String (iso format)

  `comment_description`:

    {'cid': comment_identifier, 'lang': comment_language, 'comment': comment_body, 'uid': author_id, 'cdate': creation_date}

  * `comment_identifier`: String
  * `comment_language`: String
  * `comment_body`: String
  * `author_id`: String
  * `creation_date`: String (iso format)

##5. Get user profile

    [Base URL]/profile/[lang]
    [Base URL]/profile/[lang]/[uid]

  * `lang`: language string
  * `uid`: String or not specified (authorized user uid assumed}

###Responses:

Response JSON:

    {'uid': user_id, 'first_name': first_name, 'last_name': last_name, 'profile': profile_description, 'questions': qid_list, 'answers': aid_list, 'faces': face_obj}

  * `user_id`: String
  * `first_name`: String
  * `last_name`: String
  * `profile_description`: {...}
  * `qid_list`: Array [String]
  * `aid_list`: Array [String]
  * `face_obj`: Object of different face sizes {50: '...', 100: '...'}

##6. Comment question

Allows to comment questions in the name of authorized user

Method: *POST*

    [Base URL]/comment/question/[qid]?[parameters]

  * `qid`: question identifier
  * `parameters`:
    * `body`: comment body (text/plain)
    * `lang` (optional): message translation language (defaults to question language)

###Responses:

Response JSON:

    {'status': 'ok', 'body': sanitized_comment_body, 'cid': comment_identifier, 'qid': question_identifier, 'cdate': creation_date}

  * `sanitized_comment_body`: String
  * `comment_identifier`: String
  * `question_identifier`: String
  * `creation_date`: String (iso format, eg.: '2011-09-06T14:31:38.858121Z')

The question was successfully commented.

Response 403:

The body is too large or empty.

##7. Comment answer

Allows to comment answer in the name of authorized user

Method: *POST*

    [Base URL]/comment/answer/[aid]?[parameters]

  * `aid`: answer identifier
  * `parameters`:
    * `body`: comment body (text/plain)
    * `lang` (optional): message translation language (defaults to answer language)

###Responses:

Response JSON:

    {'status': 'ok', 'body': sanitized_comment_body, 'cid': comment_identifier, 'aid': answer_identifier, 'cdate': creation_date}

  * `sanitized_comment_body`: String
  * `comment_identifier`: String
  * `answer_identifier`: String
  * `creation_date`: String (iso format, eg.: '2011-09-06T14:31:38.858121Z')

The answer was successfully commented.

Response 403:

The body is too large or empty.
