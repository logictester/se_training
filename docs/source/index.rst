.. SE Training - Social Login documentation master file, created by
   sphinx-quickstart on Thu Feb 13 16:24:12 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

UJO - Social Login
##################

.. toctree::
   :maxdepth: 3
   :hidden:

   index

Admin UI
********

   1. Launch Admin UI: `Admin UI Console <https://console.tryciam.onewelcome.io/>`_
   2. Select the training tenant
   3. Identity Broker -> Identity Providers -> Add identity provider -> OpenID Connect
   4. Use the client/secret for all
   5. Load the well-known URL into the form: 

      .. code-block::

        https://accounts.google.com/.well-known/openid-configuration

   6. Add variant 

      .. note::
         
         Variant name: **AUTHENTICATION**

         Scope names: **openid profile email**

      .. image:: /img/image1.png
   
   7. Submit


Identity Broker Tulip Client configuration
******************************************

   1. Generate an ID Broker Access Token in Postman

      .. image:: /img/image2.png

   2. Generate a client secret (must be 32 character long): `Secret Generator <https://passwords-generator.org/32-character>`_
   3. Replace the redirectUri with your POD URI in the following format: **https://productpod-se-training-name-deployment.tryciam.onewelcome.io/training/login?return_from=Idbgoogle**

      .. image:: /img/image3.png 
   4. Send


OpenAM
******

   1. Open K9s and port forward openam

      .. caution::

        Add the openam URL to your hosts file mapped to 127.0.0.1

   2. amadmin + where to find password?? (application.secrets.yaml)
   3. Top Realm -> Authentication -> Modules
   4. Add Module

   5. 
   6. Configs - use tulip as client and the secret generated above

   7. 
   
   8. Add OpenDJ class data: Top Realm -> Data Stores -> OpenDJ
   9. LDAP User Object Class, add objectidbgoogleId
   10. 

   11. LDAP User Attributes, add idbgoogleId
   12. 

Tulip YAML configurations
*************************

  .. danger::

    Please pay extra attention to indentation when working with YAML files. Incorrect indentation will cause errors and issues even if the build succeeds

1. config/demo/application.yaml add schema under iwelcome.schema.thales
=======================================================================
   
      .. code:: JSON

        {
          "schema": "urn:scim:schemas:extension:iwelcome:1.0:idbgoogle",
          "subAttributes": [],
          "primary": false,
          "name": "idbgoogleId",
          "type": "string",
          "multiValued": false,
          "description": "Holds value of user's IDB Google ID",
          "readOnly": false,
          "required": false,
          "caseExact": true,
          "unique": true,
          "openamUserAttribute": {
            "_id": null,
            "attributeTypeOID": "1.3.6.1.4.1.44444.1.1.1.21",
            "objectClassOID": "1.3.6.1.4.1.44444.100.1.1.21",
            "name": "idbgoogleId",
            "identifying": false
          },
          "idbgoogleMapping": "id",
          "avmActive": false
        },

2. config/demo/credential.yaml:
===============================
    
    .. code::

      iwelcome.opendj.ldap.objectclasses[increase the number]: objectidbgoogleId

    .. code::
      
      iwelcome.opendj.ldap.mappings[increase the number].from: '[''idbgoogleId'']'
      iwelcome.opendj.ldap.mappings[same number].to: idbgoogleId
      
      
    .. code:: JSON
      
      iwelcome.socialLinking.thales.idbgoogle.linkTemplateScim: |-
        {
          "schemas": [
            "urn:scim:schemas:core:1.0",
            "urn:scim:schemas:extension:iwelcome:1.0",
            "urn:scim:schemas:extension:iwelcome:1.0:idbgoogle",
            "urn:scim:schemas:extension:iwelcomeattributevaluemetadata:1.0"
          ],
          "urn:scim:schemas:extension:iwelcome:1.0:idbgoogle": {
            "idbgoogleId": "#userMap['id']"
          }
        }
      iwelcome.socialLinking.thales.idbgoogle.profileMapping: |-
        {
          "id": "#socialUser['sub']"
        }
      iwelcome.socialLinking.thales.idbgoogle.unlinkTemplateScim: |-
        {
          "schemas": [
            "urn:scim:schemas:core:1.0",
            "urn:scim:schemas:extension:iwelcome:1.0:idbgoogle"
          ],
          "meta": {
            "attributes": [
              "urn:scim:schemas:extension:iwelcome:1.0:idbgoogle:idbgoogleId"
            ]
          }
        }
      
      
    .. code:: JSON
      
      iwelcome.socialProviders.thales.idbgoogle.clientId: "provided Google client ID"
      iwelcome.socialProviders.thales.idbgoogle.clientSecret: "provided Google client secret"
      iwelcome.socialProviders.thales.idbgoogle.enabled: "true"
      iwelcome.socialProviders.thales.idbgoogle.grantType: authorization_code
      iwelcome.socialProviders.thales.idbgoogle.profileFields: id
      iwelcome.socialProviders.thales.idbgoogle.redirectUri: https://productpod-se-training-name-deployment.tryciam.onewelcome.io/training/profile/security/google
      iwelcome.socialProviders.thales.idbgoogle.responseType: code
      iwelcome.socialProviders.thales.idbgoogle.scope: email profile openid
      iwelcome.socialProviders.thales.idbgoogle.wellKnownApiUrl: https://accounts.google.com/.well-known/openid-configuration
      
3. config/demo/login-api.yaml:
==============================

       iwelcome.workflow.social.workflow-base-url.thales.training: https://productpod-se-training-name-deployment.tryciam.onewelcome.io/training
      
   
      iwelcome.loginModules.socialOptions: |-
          [
            {
              "moduleName": "IdbgoogleSocialAuthentication",
              "providerName": "idbgoogle"
            }
          ]
   
      
      
      Under:
      
      
       iwelcome.loginModules.openam.thales: |-
      
      Add:
      
           {
             "moduleName": "IdbgoogleSocialAuthentication",
             "authType": "module",
             "authName": "IdbgoogleSocialAuthentication",
             "payload": {
               "authId": "%s",
               "state": "%s",
               "code": "%s"
             }
           },
      
      Under:
      
      iwelcome.loginModules.api.thales.training: |-
      
      Add:
      
           {
             "moduleName": "IdbgoogleSocialAuthentication",
             "requiredFields": [
               "state",
               "code"
             ],
             "uri": "idbgoogle",
             "screenToUse": "Version1"
           },
      
   1. config/demo/opendj-userstore.yaml
   Add:
   
   schema: |
     dn: cn=schema
     objectClass: top
     objectClass: ldapSubentry
     objectClass: subschema
     cn: schema
     attributeTypes: ( 1.3.6.1.4.1.44444.1.1.1.21 NAME 'idbgoogleId' EQUALITY caseIgnoreMatch ORDERING caseIgnoreOrderingMatch SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 USAGE userApplications )
     objectClasses: ( 1.3.6.1.4.1.44444.100.1.1.21 NAME 'objectidbgoogleId' SUP top AUXILIARY MAY (idbgoogleId) )
   
   2. config/demo/workflowapi.secrets.yaml - decode -> add -> encode

            "find_user_by_email_2": {
                "request": {
                    "method": "GET",
                    "endpoint": "http://ums:8091/v2/Users/?filter=emails eq \"{email}\"",
                    "urlPlaceholders": {
                        "email": "#execution['user']['emails']"
                    }
                },
                "responseContainer": "internalUser"
            },

   5. config/demo/workflowapi.yaml

   Under: 
   
       iwelcome.scim.postObject.thales.training: |-
   
   Add:
   
   
           "create_link_user_task": {
             "segment": "'thales'",
             "brand": "'training'",
             "locale": "'en_GB'",
             "operation": "'post1'",
             "values": {
               "emails": [
                 {
                   "primary": true,
                   "type": "'home'",
                   "value": "#userMap['emails']"
                 }
               ],
               "preferredLanguage": "'en_GB'",
               "name": {
                 "familyName": "#userMap['familyName']",
                 "givenName": "#userMap['givenName']"
               },
               "schemas": [
                 "urn:scim:schemas:core:1.0",
                 "urn:scim:schemas:extension:iwelcome:1.0",
                 "urn:scim:schemas:extension:iwelcome:1.0:ritm"
               ],
               "urn:scim:schemas:extension:iwelcome:1.0": {
                 "segment": "'thales'",
                 "state": "'ACTIVE'"
               },
               "urn:scim:schemas:extension:iwelcome:1.0:ritm": {
                 "groups": [
                   {
                     "structureCode": "'default'",
                     "itemCode": "'default'"
                   }
                 ],
                 "roles": [
                   {
                     "role": "'personal'",
                     "startDate": "'1970-01-01T00:00:00.000Z'",
                     "endDate": "'2100-01-01T00:00:00.000Z'"
                   }
                 ]
               }
             }
           }
   

   6. kubefiles/demo/uic/themes/thales-training-wfe-iwmuitheme.yaml


            "IdbgoogleLoginButton": {
              "labelContent": {
                  "alignItems": "center",
                  "display": "inline-flex",
                  "justifyContent": "center",
                  "minWidth": "200px",
                  "flexDirection": "row-reverse",
                  "gap": "5px"
              },
              "icon": {
                "& img": {
                  "width": "22px !important",
                  "height": "22px !important"
                },
                "width": "32px !important",
                "height": "22px !important"
              },
              "root": {
                "&:hover": {
                  "backgroundColor": "#172F55!important",
                  "opacity": 0.8
                },
                "backgroundColor": "#172F55!important",
                "color": "#fff!important"
              }
            },

      
   7. kubefiles/demo/uic/ui/thales.yaml 
   
   Under: 
   
   ui/thales.training.page.login: |-
   
   Add:
      
            "idbgoogle": {
              "bottomDescription": true,
              "bottomDescriptionLink": "",
              "enable": true,
              "iconDetails": {
                "type": "file",
                "value": "https://productpod-se-training-name.tryciam.onewelcome.io/training/login/ui/resources/theme/img/google.png"
              },
              "label": "buttonsLabel.idbgoogle.label",
              "stylesKey": [
                "LoginSocialButtons",
                "LoginTwoFASmsClasses",
                "IdbgoogleLoginButton"
              ],
              "topDescription": false,
              "topDescriptionLink": "",
              "url": ""
            },

   Under:
   
   ui/thales.training.page.security: |-
   
   Under Passkey add:
   
   
                "cards": {
                  "idbgoogle": {
                      "enable": true,
                      "iconDetails": {
                      "type": "file",
                      "value": "logo-google.png"
                      },
                      "linkButtonLabel": "buttonsLabel.link",
                      "name": "google",
                      "provider": "Google",
                      "stylesKey": [
                      "SecurityCardClasses"
                      ],
                      "title": "security.google.title",
                      "unlinkButtonLabel": "buttonsLabel.unlink"
                    }
                  },
                  "description": "security.container.social_provider.description",
                  "enableToolTip": false,
                  "label": "security.changePassword",
                  "subtype": "social_provider",
                  "tooltipIcon": "info",
                  "type": "CARDS_CONTAINER"
              },
   
   
   Add pages:
   
   
       ui/thales.training.workflowEngine.link_account: |-
         {
           "configuration": {
             "components": [
               {
                 "defaultProps": {
                   "iconElementLeft": {
                     "defaultProps": {
                       "target": "_blank",
                       "to": "/training/login"
                     }
                   },
                   "iconElementRight": {
                     "menuItems": [
                       {
                         "defaultProps": {
                           "primaryText": "Deutsch",
                           "value": "de"
                         },
                         "lang": "de_DE"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "English",
                           "value": "en"
                         },
                         "lang": "en_GB"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Español",
                           "value": "es"
                         },
                         "lang": "es_ES"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Français",
                           "value": "fr"
                         },
                         "lang": "fr_FR"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Italiano",
                           "value": "it"
                         },
                         "lang": "it_IT"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Nederlands",
                           "value": "nl"
                         },
                         "lang": "nl_NL"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Română",
                           "value": "ro"
                         },
                         "lang": "ro_RO"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Svenska",
                           "value": "sv"
                         },
                         "lang": "sv_SE"
                       }
                     ]
                   },
                   "stylesKey": "HeaderClasses"
                 },
                 "subtype": "Header",
                 "type": "HEADER"
               },
               {
                 "defaultProps": {
                   "stylesKey": [
                     "FormTitleWrapperClasses"
                   ],
                   "title": "link_account.title"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "type": "ERROR_TEXT"
               },
               {
                 "defaultProps": {
                   "style": {
                     "color": "#272727",
                     "textAlign": "left"
                   },
                   "stylesKey": [
                     "TextClasses",
                     "LoginTextClasses"
                   ],
                   "text": "link_account.redirect.text1"
                 },
                 "type": "TEXT"
               },
               {
                 "defaultProps": {
                   "style": {
                     "color": "#272727",
                     "textAlign": "left"
                   },
                   "stylesKey": [
                     "TextClasses",
                     "LoginTextClasses"
                   ],
                   "text": "link_account.redirect.text2"
                 },
                 "type": "TEXT"
               },
               {
                 "defaultProps": {
                   "style": {
                     "color": "#272727",
                     "textAlign": "left"
                   },
                   "stylesKey": [
                     "TextClasses",
                     "LoginTextClasses"
                   ],
                   "text": "link_account.redirect.text3"
                 },
                 "type": "TEXT"
               },
               {
                 "components": [
                   {
                     "defaultProps": {
                       "alignCenter": false,
                       "label": "link_account.continueButton",
                       "name": "continueButton",
                       "requestSettings": {
                         "extraPayloadParams": [
                           {
                             "name": "option",
                             "value": true
                           }
                         ]
                       },
                       "style": {
                         "marginTop": "40px",
                         "boxShadow": "0px 1px 5px 0px rgba(0,0,0,0.2), 0px 2px 2px 0px rgba(0,0,0,0.14), 0px 3px 1px -2px rgba(0,0,0,0.12)"
                       },
                       "stylesKey": [
                         "ConsentSubmitButtonClasses",
                         "ComponentRowClasses"
                       ]
                     },
                     "formName": "mainForm",
                     "handlers": [
                       {
                         "action": "logoutDevices",
                         "propName": "onClick"
                       }
                     ],
                     "isValid": true,
                     "name": "option",
                     "type": "SUBMIT_BUTTON"
                   },
                   {
                     "defaultProps": {
                       "label": "link_account.redirectButton",
                       "redirectUrl": "https://productpod-se-training-name.tryciam.onewelcome.io/training/login",
                       "style": {
                         "marginTop": "40px",
                         "color": "#272727",
                         "backgroundColor": "white",
                         "boxShadow": "0px 1px 5px 0px rgba(0,0,0,0.2), 0px 2px 2px 0px rgba(0,0,0,0.14), 0px 3px 1px -2px rgba(0,0,0,0.12)"
                       },
                       "stylesKey": [
                         "SubmitButtonClasses",
                         "DemoFooterButtonClasses"
                       ]
                     },
                     "name": "redirectButton",
                     "type": "REDIRECT_BUTTON"
                   }
                 ],
                 "mainClassname": "submitButtons",
                 "type": "FORM"
               }
             ],
             "mainAppBodyClassName": "loginAppBody",
             "mainAppContentClassName": "loginAppContent",
             "mainClassname": "loginForm",
             "type": "FORM",
             "disableSubmit": true
           },
           "name": "link_account",
           "title": "link_account.pageTitle",
           "type": "workflowEngine"
         }
       ui/thales.training.workflowEngine.profile_completion_step_failed: |-
         {
           "configuration": {
             "components": [
               {
                 "defaultProps": {
                   "iconElementLeft": {
                     "defaultProps": {
                       "target": "_blank",
                       "to": "/training/login/"
                     }
                   },
                   "iconElementRight": {
                     "menuItems": [
                       {
                         "defaultProps": {
                           "primaryText": "Deutsch",
                           "value": "de"
                         },
                         "lang": "de_DE"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "English",
                           "value": "en"
                         },
                         "lang": "en_GB"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Español",
                           "value": "es"
                         },
                         "lang": "es_ES"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Français",
                           "value": "fr"
                         },
                         "lang": "fr_FR"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Italiano",
                           "value": "it"
                         },
                         "lang": "it_IT"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Nederlands",
                           "value": "nl"
                         },
                         "lang": "nl_NL"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Română",
                           "value": "ro"
                         },
                         "lang": "ro_RO"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Svenska",
                           "value": "sv"
                         },
                         "lang": "sv_SE"
                       }
                     ]
                   },
                   "stylesKey": "HeaderClasses"
                 },
                 "subtype": "Header",
                 "type": "HEADER"
               },
               {
                 "defaultProps": {
                   "stylesKey": [
                     "FormTitleWrapperClasses",
                     "LoginFormTitleClasses",
                     "ErrorFormTitleClasses"
                   ],
                   "title": "profile_completion_step_failed.title"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "defaultProps": {
                   "stylesKey": [
                     "TextClasses",
                     "LoginTextClasses"
                   ],
                   "text": "profile_completion_step_failed.description"
                 },
                 "type": "TEXT"
               },
               {
                 "type": "TEXT",
                 "defaultProps": {
                   "stylesKey": [
                     "TextClasses",
                     "LoginTextClasses"
                   ],
                   "text": "whitespace",
                   "style": {
                     "textAlign": "left",
                     "color": "#333333",
                     "fontSize": "0em"
                   }
                 }
               },
               {
                 "defaultProps": {
                   "label": "404.redirectButton",
                   "redirectUrl": "https://productpod-se-training-name.tryciam.onewelcome.io/training/login",
                   "stylesKey": [
                     "SubmitButtonClasses"
                   ]
                 },
                 "name": "redirectButton",
                 "type": "REDIRECT_BUTTON"
               }
             ],
             "mainAppBodyClassName": "loginAppBody",
             "mainAppContentClassName": "loginAppContent",
             "mainClassname": "loginForm",
             "type": "FORM",
             "disableSubmit": true
           },
           "name": "profile_completion_step_failed",
           "title": "profile_completion_step_failed.pageTitle",
           "type": "workflowEngine"
         }
         {
           "configuration": {
             "components": [
               {
                 "defaultProps": {
                   "iconElementLeft": {
                     "defaultProps": {
                       "target": "_blank",
                       "to": "/b2b/login"
                     }
                   },
                   "iconElementRight": {
                     "menuItems": [
                       {
                         "defaultProps": {
                           "primaryText": "Deutsch",
                           "value": "de"
                         },
                         "lang": "de_DE"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "English",
                           "value": "en"
                         },
                         "lang": "en_GB"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Español",
                           "value": "es"
                         },
                         "lang": "es_ES"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Français",
                           "value": "fr"
                         },
                         "lang": "fr_FR"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Italiano",
                           "value": "it"
                         },
                         "lang": "it_IT"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Nederlands",
                           "value": "nl"
                         },
                         "lang": "nl_NL"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Română",
                           "value": "ro"
                         },
                         "lang": "ro_RO"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Svenska",
                           "value": "sv"
                         },
                         "lang": "sv_SE"
                       }
                     ]
                   },
                   "stylesKey": "HeaderClasses"
                 },
                 "subtype": "Header",
                 "type": "HEADER"
               },
               {
                 "defaultProps": {
                   "stylesKey": [
                     "FormTitleWrapperClasses"
                   ],
                   "title": "registration_selection_mfa_step_decision.title"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "type": "ERROR_TEXT"
               },
               {
                 "defaultProps": {
                   "stylesKey": [
                     "TextClasses",
                     "LoginTextClasses"
                   ],
                   "text": "registration_selection_mfa_step_decision.text"
                 },
                 "type": "TEXT"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "label": "registration_selection_mfa_step_decision.totpButton",
                   "name": "totpButton",
                   "requestSettings": {
                     "extraPayloadParams": [
                       {
                         "name": "option",
                         "value": "totp"
                       }
                     ]
                   },
                   "stylesKey": [],
                   "style": {
                     "backgroundImage": "url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 24 24\"><path d=\"M19 1H9c-1.1 0-2 .9-2 2v3h2V4h10v16H9v-2H7v3c0 1.1.9 2 2 2h10c1.1 0 2-.9 2-2V3c0-1.1-.9-2-2-2zm-8.2 10V9.5C10.8 8.1 9.4 7 8 7S5.2 8.1 5.2 9.5V11c-.6 0-1.2.6-1.2 1.2v3.5c0 .7.6 1.3 1.2 1.3h5.5c.7 0 1.3-.6 1.3-1.2v-3.5c0-.7-.6-1.3-1.2-1.3zm-1.3 0h-3V9.5c0-.8.7-1.3 1.5-1.3s1.5.5 1.5 1.3V11z\"></path></svg>')",
                     "backgroundRepeat": "no-repeat",
                     "backgroundPosition": "40px center",
                     "backgroundSize": "31px 31px",
                     "fontSize": "1rem",
                     "width": "100%",
                     "margin": "24px 0",
                     "display": "flex",
                     "outline": "none",
                     "padding": "12px 24px 12px 90px",
                     "minHeight": "70px",
                     "textAlign": "left",
                     "alignItems": "center",
                     "justifyContent": "flex-start",
                     "borderRadius": "4px",
                     "backgroundColor": "white",
                     "boxSizing": "border-box",
                     "borderColor": "black",
                     "--color": "#fff!important",
                     "--size": "100px",
                     "--thickness": "1.5",
                     "color": "#252A71",
                     "fontFamily": "\"Red Hat Display\", sans-serif !important",
                     "boxShadow": "0 1px 3px 0 rgba(0, 0, 0, .2), 0 1px 1px 0 rgba(0, 0, 0, .14), 0 2px 1px -1px rgba(0, 0, 0, .12)"
                   }
                 },
                 "formName": "mainForm",
                 "handlers": [
                   {
                     "action": "logoutDevices",
                     "propName": "onClick"
                   }
                 ],
                 "isValid": true,
                 "name": "option",
                 "type": "SUBMIT_BUTTON"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "label": "registration_selection_mfa_step_decision.smsButton",
                   "name": "emailButton",
                   "requestSettings": {
                     "extraPayloadParams": [
                       {
                         "name": "option",
                         "value": "sms"
                       }
                     ]
                   },
                   "stylesKey": [
                     "IconClasses",
                     "LoginTwoFASmsClasses"
                   ],
                   "style": {
                     "backgroundImage": "url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 24 24\"><path d=\"M20,2H4A2,2 0 0,0 2,4V22L6,18H20A2,2 0 0,0 22,16V4A2,2 0 0,0 20,2M6,9H18V11H6M14,14H6V12H14M18,8H6V6H18\"></path></svg>')",
                     "backgroundRepeat": "no-repeat",
                     "backgroundPosition": "40px center",
                     "backgroundSize": "31px 31px",
                     "fontSize": "1rem",
                     "width": "100%",
                     "margin": "24px 0",
                     "display": "flex",
                     "outline": "none",
                     "padding": "12px 24px 12px 90px",
                     "minHeight": "70px",
                     "textAlign": "left",
                     "alignItems": "center",
                     "justifyContent": "flex-start",
                     "borderRadius": "4px",
                     "backgroundColor": "white",
                     "boxSizing": "border-box",
                     "borderColor": "black",
                     "--color": "#fff!important",
                     "--size": "100px",
                     "--thickness": "1.5",
                     "color": "#252A71",
                     "fontFamily": "\"Red Hat Display\", sans-serif !important",
                     "boxShadow": "0 1px 3px 0 rgba(0, 0, 0, .2), 0 1px 1px 0 rgba(0, 0, 0, .14), 0 2px 1px -1px rgba(0, 0, 0, .12)"
                   }
                 },
                 "formName": "mainForm",
                 "handlers": [
                   {
                     "action": "logoutDevices",
                     "propName": "onClick"
                   }
                 ],
                 "isValid": true,
                 "name": "option",
                 "type": "SUBMIT_BUTTON"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "label": "registration_selection_mfa_step_decision.fidoButton",
                   "name": "emailButton",
                   "requestSettings": {
                     "extraPayloadParams": [
                       {
                         "name": "option",
                         "value": "fido"
                       }
                     ]
                   },
                   "stylesKey": [
                     "IconClasses",
                     "LoginTwoFASmsClasses"
                   ],
                   "style": {
                     "backgroundImage": "url('https://sandbox-us-alex-1-deployment-dev.tryciam.onewelcome.io/b2b/login/ui/resources/theme/img/passkey.png')",
                     "backgroundRepeat": "no-repeat",
                     "backgroundPosition": "40px center",
                     "backgroundSize": "31px 31px",
                     "fontSize": "1rem",
                     "width": "100%",
                     "margin": "24px 0",
                     "display": "flex",
                     "outline": "none",
                     "padding": "12px 24px 12px 90px",
                     "minHeight": "70px",
                     "textAlign": "left",
                     "alignItems": "center",
                     "justifyContent": "flex-start",
                     "borderRadius": "4px",
                     "backgroundColor": "white",
                     "boxSizing": "border-box",
                     "borderColor": "black",
                     "--color": "#fff!important",
                     "--size": "100px",
                     "--thickness": "1.5",
                     "color": "#252A71",
                     "fontFamily": "\"Red Hat Display\", sans-serif !important",
                     "boxShadow": "0 1px 3px 0 rgba(0, 0, 0, .2), 0 1px 1px 0 rgba(0, 0, 0, .14), 0 2px 1px -1px rgba(0, 0, 0, .12)"
                   }
                 },
                 "formName": "mainForm",
                 "handlers": [
                   {
                     "action": "logoutDevices",
                     "propName": "onClick"
                   }
                 ],
                 "isValid": true,
                 "name": "option",
                 "type": "SUBMIT_BUTTON"
               },
               {
                 "defaultProps": {
                   "stylesKey": "HrClasses",
                   "text": "common.or"
                 },
                 "type": "HR_TEXT"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "label": "registration_selection_mfa_step_decision.remindLater",
                   "name": "remindButton",
                   "requestSettings": {
                     "extraPayloadParams": [
                       {
                         "name": "option",
                         "value": "skip"
                       }
                     ]
                   },
                   "stylesKey": [
                     "ComponentRowClasses",
                     "LoginTwoFASmsClasses"
                   ],
                   "style": {
                     "backgroundImage": "url('data:image/svg+xml;utf8,<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 24 24\"><path d=\"M12,2C6.5,2,2,6.5,2,12s4.5,10,10,10s10-4.5,10-10S17.5,2,12,2z M16.2,16.2L11,13V7h1.5v5.2l4.5,2.7L16.2,16.2zz\"></path></svg>')",
                     "backgroundRepeat": "no-repeat",
                     "backgroundPosition": "40px center",
                     "backgroundSize": "31px 31px",
                     "fontSize": "1rem",
                     "width": "100%",
                     "margin": "24px 0 40px 0",
                     "display": "flex",
                     "outline": "none",
                     "padding": "12px 24px 12px 90px",
                     "minHeight": "70px",
                     "textAlign": "left",
                     "alignItems": "center",
                     "justifyContent": "flex-start",
                     "borderRadius": "4px",
                     "backgroundColor": "white",
                     "boxSizing": "border-box",
                     "borderColor": "black",
                     "--color": "#fff!important",
                     "--size": "100px",
                     "--thickness": "1.5",
                     "color": "#252A71",
                     "fontFamily": "\"Red Hat Display\", sans-serif !important",
                     "boxShadow": "0 1px 3px 0 rgba(0, 0, 0, .2), 0 1px 1px 0 rgba(0, 0, 0, .14), 0 2px 1px -1px rgba(0, 0, 0, .12)"
                   }
                 },
                 "formName": "mainForm",
                 "handlers": [
                   {
                     "action": "logoutDevices",
                     "propName": "onClick"
                   }
                 ],
                 "isValid": true,
                 "name": "option",
                 "type": "SUBMIT_BUTTON"
               }
             ],
             "mainAppBodyClassName": "loginAppBody",
             "mainAppContentClassName": "cardWithFooter",
             "mainClassname": "loginForm",
             "type": "FORM",
             "disableSubmit": true
           },
           "name": "registration_selection_mfa_step_decision",
           "title": "registration_selection_mfa_step_decision.pageTitle",
           "type": "workflowEngine"
         }
           "configuration": {
             "components": [
               {
                 "defaultProps": {
                   "iconElementLeft": {
                     "defaultProps": {
                       "target": "_blank",
                       "to": "/b2b/login"
                     }
                   },
                   "iconElementRight": {
                     "menuItems": [
                       {
                         "defaultProps": {
                           "primaryText": "Deutsch",
                           "value": "de"
                         },
                         "lang": "de_DE"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "English",
                           "value": "en"
                         },
                         "lang": "en_GB"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Español",
                           "value": "es"
                         },
                         "lang": "es_ES"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Français",
                           "value": "fr"
                         },
                         "lang": "fr_FR"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Italiano",
                           "value": "it"
                         },
                         "lang": "it_IT"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Nederlands",
                           "value": "nl"
                         },
                         "lang": "nl_NL"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Română",
                           "value": "ro"
                         },
                         "lang": "ro_RO"
                       },
                       {
                         "defaultProps": {
                           "primaryText": "Svenska",
                           "value": "sv"
                         },
                         "lang": "sv_SE"
                       }
                     ]
                   },
                   "stylesKey": "HeaderClasses"
                 },
                 "subtype": "Header",
                 "type": "HEADER"
               },
               {
                 "defaultProps": {
                   "stylesKey": [
                     "FormTitleWrapperClasses",
                     "LoginFormTitleClasses"
                   ],
                   "title": "show_recovery_codes_step.title"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "type": "ERROR_TEXT"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "style": {
                     "color": "#272727",
                     "textAlign": "left",
                     "margin-bottom": "20px"
                   },
                   "stylesKey": [
                     "LoginTextClasses",
                     "TextClasses"
                   ],
                   "text": "show_recovery_codes_step.description1"
                 },
                 "type": "TEXT"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "style": {
                     "color": "#272727",
                     "textAlign": "left",
                     "margin-bottom": "20px"
                   },
                   "stylesKey": [
                     "LoginTextClasses",
                     "TextClasses"
                   ],
                   "text": "show_recovery_codes_step.description2"
                 },
                 "type": "TEXT"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "style": {
                     "color": "#272727",
                     "textAlign": "left",
                     "margin-bottom": "40px"
                   },
                   "stylesKey": [
                     "LoginTextClasses",
                     "TextClasses"
                   ],
                   "text": "show_recovery_codes_step.description3"
                 },
                 "type": "TEXT"
               },
               {
                 "defaultProps": {
                   "alignCenter": false,
                   "style": {
                     "fontFamily": "'Courier New', monospace !important",
                     "display": "flex",
                     "width": "50%",
                     "margin-bottom": "10px !important"
                   },
                   "stylesKey": [
                     "TOTPFormTitleWrapperClasses"
                   ],
                   "description": "show_recovery_codes_step.codeOne"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "defaultProps": {
                   "alignCenter": false,
                   "style": {
                     "fontFamily": "'Courier New', monospace !important",
                     "display": "flex",
                     "width": "50%",
                     "margin-bottom": "10px !important"
                   },
                   "stylesKey": [
                     "TOTPFormTitleWrapperClasses"
                   ],
                   "description": "show_recovery_codes_step.codeTwo"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "defaultProps": {
                   "alignCenter": false,
                   "style": {
                     "fontFamily": "'Courier New', monospace !important",
                     "display": "flex",
                     "width": "50%",
                     "margin-bottom": "10px !important"
                   },
                   "stylesKey": [
                     "TOTPFormTitleWrapperClasses"
                   ],
                   "description": "show_recovery_codes_step.codeThree"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "defaultProps": {
                   "alignCenter": false,
                   "style": {
                     "fontFamily": "'Courier New', monospace !important",
                     "display": "flex",
                     "width": "50%",
                     "margin-bottom": "10px !important"
                   },
                   "stylesKey": [
                     "TOTPFormTitleWrapperClasses"
                   ],
                   "description": "show_recovery_codes_step.codeFour"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "defaultProps": {
                   "alignCenter": false,
                   "style": {
                     "fontFamily": "'Courier New', monospace !important",
                     "display": "flex",
                     "width": "50%",
                     "margin-bottom": "10px !important"
                   },
                   "stylesKey": [
                     "TOTPFormTitleWrapperClasses"
                   ],
                   "description": "show_recovery_codes_step.codeFive"
                 },
                 "subtype": "FormTitle",
                 "type": "FORM_TITLE"
               },
               {
                 "defaultProps": {
                   "alignCenter": true,
                   "label": "common.continue",
                   "name": "Submit",
                   "style": {
                     "margin-top": "40px",
                     "margin-bottom": "25px"
                   },
                   "stylesKey": [
                     "ComponentRowClasses",
                     "SignupButtonClasses"
                   ]
                 },
                 "formName": "mainForm",
                 "handlers": [
                   {
                     "action": "submit",
                     "propName": "onClick",
                     "validators": []
                   }
                 ],
                 "isValid": true,
                 "name": "submitbutton",
                 "type": "SUBMIT_BUTTON"
               }
             ],
             "mainAppBodyClassName": "loginAppBody",
             "mainAppContentClassName": "cardWithFooter",
             "mainClassname": "loginForm",
             "type": "FORM",
             "disableSubmit": false
           },
           "name": "show_recovery_codes_step",
           "title": "show_recovery_codes_step.pageTitle",
           "type": "workflowEngine"
         }
   
   

   8. kubefiles/demo/uic/translations/thales-training.yaml

   Under:
   
   translations/thales.training.en_GB_login: |-
   
   Add:
   
   
              "idbgoogle": {
                "authenticationLabel": "Authenticate with Google",
                "bottomDescription": "",
                "label": "Log in with",
                "topDescription": ""
              },
   
   
   Under:
   
   translations/thales.training.en_GB: |-
   
   Add:
   
   
            "idbgoogle": {
              "title": "Google account",
              "modalTitle": "Social Linking Google"
            },



            "link_account": {
              "continueButton": "$t(common.continue)",
              "title": "Link your account",
              "title_title_attribute": "Link your account",
              "redirectButton": "$t(common.cancelLabel)",
              "redirect": {
                "text1": "You are trying to log in with your social account for the first time. No problem!",
                "text2": "Click '$t(common.continue)' to setup an account with the name and email address of your social account.",
                "text3": "If you already have an $t(common.appName) account, it will be automatically linked with your social."
              },
              "pageTitle": "$t(common.login) | $t(common.appName)"
            },
      
   
   9. kubefiles/demo/workflowapi/bpmn/kvset.yaml

   Add a new section by pasting the following code:
   
   
       bpmn/thales.training.account-link.bpmn20.xml: |-
         <?xml version="1.0" encoding="UTF-8"?>
         <definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:activiti="http://activiti.org/bpmn" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.iwelcome.com/registration">
           <process id="thales.training.account-link" name="Social Linking flow" isExecutable="true">
             <startEvent id="linking_start_event" name="Start"/>
             <sequenceFlow sourceRef="linking_start_event" targetRef="generate_process_token"/>
             <serviceTask id="generate_process_token" name="generate_process_token" activiti:delegateExpression="${generateProcessTokenTask}"/>
             <sequenceFlow sourceRef="generate_process_token" targetRef="link_account_task"/>
             <userTask id="link_account_task" name="link_account"/>
             <sequenceFlow sourceRef="link_account_task" targetRef="normalize_email_task"/>
             <scriptTask id="normalize_email_task" name="normalize_email_task" scriptFormat="groovy">
               <script>
                 Map processPublicData = com.iwelcome.workflowengine.util.WorkflowEngineUtil.getProcessPublicData(execution);
                 Map user = new HashMap();
                 String socialName = execution.getVariable("moduleName");
                 final Map idpUser = execution.getVariable("idpUser");
                 if (idpUser != null) {
                   System.out.println("SOCIAL DATA ====== " + idpUser);
                   String emails;
                   String givenName;
                   String familyName;
                   if ("GoogleSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "google");
                     execution.setVariable("identityProviderIdType", "sub");
                     emails =  idpUser.get("email");
                     givenName = idpUser.get("given_name");
                     familyName = idpUser.get("family_name");
                   } else if ("MicrosoftSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "microsoft");
                     execution.setVariable("identityProviderIdType", "id");
                     emails =  idpUser.get("userPrincipalName");
                     givenName = idpUser.get("givenName");
                     familyName = idpUser.get("surname");
                   } else if ("IdbgoogleSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "idbgoogle");
                     execution.setVariable("identityProviderIdType", "sub");
                     def idToken = idpUser.get("id_token")
                     def parts = idToken.split("\\.")
                     def decodedPayload = new String(parts[1].decodeBase64(), "UTF-8")
                     def parsedPayload = new groovy.json.JsonSlurper().parseText(decodedPayload)
                     println "Parsed idpUser Payload: $parsedPayload"                  
                     idpUser.put("sub", parsedPayload.sub)
                     emails =  parsedPayload.email;
                     givenName = parsedPayload.given_name;
                     familyName = parsedPayload.family_name;
   
                     // emails =  idpUser.get("userPrincipalName");
                     // givenName = idpUser.get("givenName");
                     // familyName = idpUser.get("surname");
                   } else if ("IdbstaSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "idbsta");
                     execution.setVariable("identityProviderIdType", "sub");
                     def idToken = idpUser.get("id_token")
                     def parts = idToken.split("\\.")
                     def decodedPayload = new String(parts[1].decodeBase64(), "UTF-8")
                     def parsedPayload = new groovy.json.JsonSlurper().parseText(decodedPayload)
                     println "Parsed idpUser Payload: $parsedPayload"                  
                     idpUser.put("sub", parsedPayload.sub)
                     emails =  parsedPayload.email;
                     givenName = parsedPayload.given_name;
                     familyName = parsedPayload.family_name;
   
                     // emails =  idpUser.get("userPrincipalName");
                     // givenName = idpUser.get("givenName");
                     // familyName = idpUser.get("surname");
                   } else if ("IdbentraidSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "idbentraid");
                     execution.setVariable("identityProviderIdType", "sub");
                     def idToken = idpUser.get("id_token")
                     def parts = idToken.split("\\.")
                     def decodedPayload = new String(parts[1].decodeBase64(), "UTF-8")
                     def parsedPayload = new groovy.json.JsonSlurper().parseText(decodedPayload)
                     println "Parsed idpUser Payload: $parsedPayload"                  
                     idpUser.put("sub", parsedPayload.sub)
                     emails =  parsedPayload.email;
                     givenName = parsedPayload.given_name;
                     familyName = parsedPayload.family_name;
   
                     // emails =  idpUser.get("userPrincipalName");
                     // givenName = idpUser.get("givenName");
                     // familyName = idpUser.get("surname");
                   } else if ("IdbyahooSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "idbyahoo");
                     execution.setVariable("identityProviderIdType", "sub");
                     def idToken = idpUser.get("id_token")
                     def parts = idToken.split("\\.")
                     def decodedPayload = new String(parts[1].decodeBase64(), "UTF-8")
                     def parsedPayload = new groovy.json.JsonSlurper().parseText(decodedPayload)
                     println "Parsed idpUser Payload: $parsedPayload"                  
                     idpUser.put("sub", parsedPayload.sub)
                     emails =  parsedPayload.email;
                     givenName = parsedPayload.firstName;
                     familyName = parsedPayload.lastName;
   
                     // emails =  idpUser.get("userPrincipalName");
                     // givenName = idpUser.get("givenName");
                     // familyName = idpUser.get("surname");
                   } else if ("AuthSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "auth");
                     execution.setVariable("identityProviderIdType", "id");
                     emails =  idpUser.get("email");
                     givenName = idpUser.get("givenName");
                     familyName = idpUser.get("surname");
                   } else if ("IdmeSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "idme");
                     execution.setVariable("identityProviderIdType", "sub");
                     def idToken = idpUser.get("id_token")
                     def parts = idToken.split("\\.")
                     def decodedPayload = new String(parts[1].decodeBase64(), "UTF-8")
                     def parsedPayload = new groovy.json.JsonSlurper().parseText(decodedPayload)
                     println "Parsed idpUser Payload: $parsedPayload"                  
                     idpUser.put("sub", parsedPayload.sub)
                     emails =  parsedPayload.email;
                     givenName = parsedPayload.fname;
                     familyName = parsedPayload.lname;
                     // emails =  idpUser.get("email");
                     // givenName = idpUser.get("fname");
                     // familyName = idpUser.get("lname");
                   } else if ("AppleSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "apple");
                     execution.setVariable("identityProviderIdType", "sub");
                     emails =  idpUser.get("email");
                     givenName = idpUser.get("firstName");
                     familyName = idpUser.get("lastName");                  
                   } else if ("FacebookSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "facebook");
                     execution.setVariable("identityProviderIdType", "id");
                   } else if ("LinkedinSocialAuthentication".equals(socialName)) {
                     execution.setVariable("identityProviderName", "linkedin");
                     execution.setVariable("identityProviderIdType", "id");
                     givenName = idpUser.get("localizedFirstName");
                     familyName = idpUser.get("localizedLastName");
                   }
                   user.put("emails", emails);
                   user.put("givenName", givenName);
                   user.put("familyName", familyName);
                   execution.setVariable("user", user);
                   System.out.println("USER DATA  ====== "+ user);
                   Map processPublicData2 = new HashMap();
                   processPublicData2.put("user", user);
                   execution.setVariable("processPublicData", processPublicData2);
                 }
               </script>
             </scriptTask>
             <sequenceFlow sourceRef="normalize_email_task" targetRef="identity_provider_gateway_1"/>
             <exclusiveGateway id="identity_provider_gateway_1"/>
             <sequenceFlow id="selected_linkedin" sourceRef="identity_provider_gateway_1" targetRef="http_request_get_linkedin_email_account_link">
               <conditionExpression xsi:type="tFormalExpression"><![CDATA[${execution.getVariable('moduleName') == 'LinkedinSocialAuthentication'}]]></conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="selected_facebook" sourceRef="identity_provider_gateway_1" targetRef="http_request_get_facebook_data">
               <conditionExpression xsi:type="tFormalExpression"><![CDATA[${execution.getVariable('moduleName') == 'FacebookSocialAuthentication'}]]></conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="selected_other" sourceRef="identity_provider_gateway_1" targetRef="http_request">
               <conditionExpression xsi:type="tFormalExpression"><![CDATA[${execution.getVariable('moduleName') != 'LinkedinSocialAuthentication'}]]></conditionExpression>
             </sequenceFlow>
             <serviceTask id="http_request_get_linkedin_email_account_link" activiti:delegateExpression="${httpRequestorTask}">
               <extensionElements>
                 <activiti:field name="httpRequestConfigName" stringValue="get_linkedin_email_account_link"/>
               </extensionElements>
             </serviceTask>
             <sequenceFlow sourceRef="http_request_get_linkedin_email_account_link" targetRef="http_request_get_linkedin_email_account_link_response_handler"/>
             <scriptTask id="http_request_get_linkedin_email_account_link_response_handler" name="abc" scriptFormat="groovy">
               <script>
                 import com.iwelcome.workflowengine.domain.http.HttpResponseData;
                 final Map getLinkedinResponse = (Map) execution.getVariable("linkedinEmail").getResponse();
                 println("======== LINKEDIN RESPONSE ==========" + getLinkedinResponse);
                 Map data = (Map) execution.getVariable("user");
                 if (getLinkedinResponse != null) {
                   data.put("emails", getLinkedinResponse.get('elements').get(0).get('handle~').get('emailAddress'));
                   execution.setVariable("user", data);
                   System.out.println("USER DATA AFTER LINKEDIN  ====== " + user);
                 }
               </script>
             </scriptTask>
             <sequenceFlow sourceRef="http_request_get_linkedin_email_account_link_response_handler" targetRef="http_request"/>
             <serviceTask id="http_request_get_facebook_data" activiti:delegateExpression="${httpRequestorTask}">
               <extensionElements>
                 <activiti:field name="httpRequestConfigName" stringValue="get_facebook_data"/>
               </extensionElements>
             </serviceTask>
             <sequenceFlow sourceRef="http_request_get_facebook_data" targetRef="http_request_get_facebook_data_response_handler"/>
             <scriptTask id="http_request_get_facebook_data_response_handler" name="abc" scriptFormat="groovy">
               <script>
                 import com.iwelcome.workflowengine.domain.http.HttpResponseData;
                 final Map getFacebookResponse = (Map) execution.getVariable("commonDataFacebook").getResponse();
                 println("======== FACEBOOK RESPONSE ==========" + getFacebookResponse);
                 Map data = (Map) execution.getVariable("user");
                 if (getFacebookResponse != null) {
                   data.put("emails", getFacebookResponse.get('email'));
                   data.put("givenName", getFacebookResponse.get('first_name'));
                   data.put("familyName", getFacebookResponse.get('last_name'));
                   execution.setVariable("user", data);
                   System.out.println("USER DATA AFTER FACEBOOK  ====== "+ user);
                 }
               </script>
             </scriptTask>
             <sequenceFlow sourceRef="http_request_get_facebook_data_response_handler" targetRef="http_request"/>
             <serviceTask id="http_request" activiti:delegateExpression="${httpRequestorTask}">
               <extensionElements>
                 <activiti:field name="httpRequestConfigName" stringValue="find_user_by_email_2"/>
               </extensionElements>
             </serviceTask>
             <sequenceFlow sourceRef="http_request" targetRef="user_found_decision_task"/>
             <scriptTask id="user_found_decision_task" name="user_found_decision_task" scriptFormat="groovy">
               <script>
                 import com.iwelcome.workflowengine.domain.http.HttpResponseData;
                 final Map internalUser = (Map) execution.getVariable("internalUser").getResponse();
                 List resouces = (List)internalUser.get("Resources");
                 if (resouces.size() == 1) {
                   execution.setVariable("isStepSuccessful", true);
                 } else {
                   execution.setVariable("isStepSuccessful", false);
                 }
               </script>
             </scriptTask>
             <sequenceFlow sourceRef="user_found_decision_task" targetRef="user_found_decision"/>
             <exclusiveGateway id="user_found_decision" name="user_found_gateway"/>
             <sequenceFlow id="is_user_found_flow" sourceRef="user_found_decision" targetRef="init_social_account">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="is_user_not_found_flow" sourceRef="user_found_decision" targetRef="check_required_data">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == false}</conditionExpression>
             </sequenceFlow>
             <serviceTask id="check_required_data" activiti:delegateExpression="${findMissingRequiredDataTask}">
               <extensionElements>
                 <activiti:field name="requiredDataConfigName" stringValue="required_insurgroup"/>
               </extensionElements>
             </serviceTask>
             <sequenceFlow sourceRef="check_required_data" targetRef="additional_data_required"/>
             <exclusiveGateway id="additional_data_required"/>
             <sequenceFlow id="data_required" sourceRef="additional_data_required" targetRef="profile_completion_step">
               <conditionExpression xsi:type="tFormalExpression">${requiredAttributesMissing == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="data_not_required" sourceRef="additional_data_required" targetRef="http_request_find_by_user_name">
               <conditionExpression xsi:type="tFormalExpression">${requiredAttributesMissing == false}</conditionExpression>
             </sequenceFlow>
             <userTask id="profile_completion_step" name="profile_completion_step">
               <extensionElements>
                 <activiti:taskListener event="create" delegateExpression="${getProcessingPurposesListener}">
                   <activiti:field name="filter" stringValue="tags eq profile_completion_step"/>
                 </activiti:taskListener>
                 <activiti:taskListener event="complete" delegateExpression="${collectProcessingPurposesListener}"/>
               </extensionElements>
             </userTask>
             <sequenceFlow sourceRef="profile_completion_step" targetRef="remove_missing_attributes"/>
             <scriptTask id="remove_missing_attributes" scriptFormat="groovy">
               <script>
                 Map publicData = execution.getVariable("processPublicData")
                 publicData.remove("missingData")
               </script>
             </scriptTask>
             <sequenceFlow sourceRef="remove_missing_attributes" targetRef="http_request_find_by_user_name"/>
             <serviceTask id="http_request_find_by_user_name" activiti:delegateExpression="${httpRequestorTask}">
               <extensionElements>
                 <activiti:field name="httpRequestConfigName" stringValue="find_user_by_user_name"/>
               </extensionElements>
             </serviceTask>
             <sequenceFlow sourceRef="http_request_find_by_user_name" targetRef="http_request_find_by_user_name_script"/>
             <scriptTask id="http_request_find_by_user_name_script" name="user_found_decision_task" scriptFormat="groovy">
               <script>
                 import com.iwelcome.workflowengine.domain.http.HttpResponseData;
                 final Map internalUser = (Map) execution.getVariable("internalUser").getResponse();
                 List resouces = (List)internalUser.get("Resources");
                 if (resouces.size() == 1) {
                   execution.setVariable("isStepSuccessful", true);
                 } else {
                   execution.setVariable("isStepSuccessful", false);
                 }
               </script>
             </scriptTask>
             <sequenceFlow sourceRef="http_request_find_by_user_name_script" targetRef="user_found_by_user_name_decision"/>
             <exclusiveGateway id="user_found_by_user_name_decision" name="user_found_gateway"/>
             <sequenceFlow id="is_user_found_by_user_name" sourceRef="user_found_by_user_name_decision" targetRef="init_social_account">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="is_user_not_found_by_user_name" sourceRef="user_found_by_user_name_decision" targetRef="create_link_user_task">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == false}</conditionExpression>
             </sequenceFlow>
             <serviceTask id="create_link_user_task" name="registration_step5" activiti:delegateExpression="${createScimUserTask}"/>
             <sequenceFlow id="step_from_scim_create_to_decision" sourceRef="create_link_user_task" targetRef="scim_create_decision"/>
             <exclusiveGateway id="scim_create_decision" name="scim_create_decision_gateway"/>
             <sequenceFlow id="scim_create_successful" sourceRef="scim_create_decision" targetRef="init_social_account">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="scim_create_failed" sourceRef="scim_create_decision" targetRef="create_account_failed">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == false}</conditionExpression>
             </sequenceFlow>
             <userTask id="create_account_failed" name="profile_completion_step_failed"/>
             <sequenceFlow sourceRef="create_account_failed" targetRef="social_linking_end_event"/>
             <serviceTask id="init_social_account" activiti:delegateExpression="${initIdentityLinkTask}"> </serviceTask>
             <sequenceFlow sourceRef="init_social_account" targetRef="init_account_decision"/>
             <exclusiveGateway id="init_account_decision" name="init_account_gateway"/>
             <sequenceFlow id="is_pending_state" sourceRef="init_account_decision" targetRef="activate_social_account">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="is_not_in_pending_state" sourceRef="init_account_decision" targetRef="last_step">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == false}</conditionExpression>
             </sequenceFlow>
             <serviceTask id="activate_social_account" activiti:delegateExpression="${activateIdentityLinkTask}"> </serviceTask>
             <sequenceFlow sourceRef="activate_social_account" targetRef="activate_social_account_gateway"/>
             <exclusiveGateway id="activate_social_account_gateway" name="activate_social_account_gateway"/>
             <sequenceFlow id="is_link_activated" sourceRef="activate_social_account_gateway" targetRef="generate_login_token">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="is_link_not_activated" sourceRef="activate_social_account_gateway" targetRef="link_account_task">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == false}</conditionExpression>
             </sequenceFlow>
             <serviceTask id="generate_login_token" name="generate_login_token" activiti:delegateExpression="${generateLoginTokenTask}"/>
             <sequenceFlow sourceRef="generate_login_token" targetRef="login_token_validation_gateway"/>
             <exclusiveGateway id="login_token_validation_gateway" name="login_token_validation_gateway"/>
             <sequenceFlow id="is_valid_login_token_flow" sourceRef="login_token_validation_gateway" targetRef="generate_consent">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == true}</conditionExpression>
             </sequenceFlow>
             <sequenceFlow id="is_not_valid_login_token_flow" sourceRef="login_token_validation_gateway" targetRef="social_linking_end_event">
               <conditionExpression xsi:type="tFormalExpression">${isStepSuccessful == false}</conditionExpression>
             </sequenceFlow>
             <!-- Generate consent -->
             <serviceTask id="generate_consent" name="generate_consent" activiti:delegateExpression="${generateConsentTask}"/>
             <sequenceFlow sourceRef="generate_consent" targetRef="generate_document_consent"/>
             <serviceTask id="generate_document_consent" name="generate_document_consent" activiti:delegateExpression="${generateDocumentConsentTask}"/>
             <sequenceFlow sourceRef="generate_document_consent" targetRef="last_step"/>
             <!-- END Generate consent -->
             <userTask id="last_step" name="thanks"/>
             <sequenceFlow sourceRef="last_step" targetRef="social_linking_end_event"/>
             <endEvent id="social_linking_end_event" name="End">
               <extensionElements>
                 <activiti:executionListener event="end" delegateExpression="${processEndEventListener}"/>
               </extensionElements>
             </endEvent>
           </process>
         </definitions>
   
   10. Save, Commit and Push

