# .Net Core Notes

#### How to handle input parameter validation for an .Net Core API request

This is to customise the validation error send by .Net Core with a custom error.

  ```
    public void ConfigureServices(IServiceCollection services)
    {
        services
            .AddControllers()
            .ConfigureApiBehaviorOptions(options => // handle input validation
            {
                options.InvalidModelStateResponseFactory = context =>
                {
                    Dictionary<string, List<string>> validationErrors = new Dictionary<string, List<string>>();

                    foreach(string field in context.ModelState.Keys)
                    {
                        context.ModelState.TryGetValue(field, out ModelStateEntry modelStateEntry);

                        foreach (ModelError modelError in modelStateEntry.Errors)
                        {
                            if(!validationErrors.ContainsKey(field))
                            {
                                validationErrors.Add(field, new List<string>());
                            }

                            if (validationErrors.TryGetValue(field, out List<string> errors))
                            {
                                errors.Add(modelError.ErrorMessage);
                            }
                        }
                    }

                    BadRequestObjectResult result = new BadRequestObjectResult(validationErrors);

                    result.ContentTypes.Add(MediaTypeNames.Application.Json);
                    result.ContentTypes.Add(MediaTypeNames.Application.Xml);

                    /*
                      * output - ex.
                      * {
                      *      "Id": [
                      *          "The Id field is required."
                      *      ]
                      * }
                      */
                    return result;
                };
            }); 
    }
  ```