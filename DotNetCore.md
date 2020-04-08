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
                    Dictionary<string, string> validationErrors = new Dictionary<string, string>();

                    foreach(string key in context.ModelState.Keys)
                    {
                        context.ModelState.TryGetValue(key, out ModelStateEntry modelStateEntry);
                        foreach (ModelError modelError in modelStateEntry.Errors)
                        {
                            validationErrors.Add(key, modelError.ErrorMessage);
                        }
                    }

                    var result = new BadRequestObjectResult(validationErrors);

                    result.ContentTypes.Add(MediaTypeNames.Application.Json);
                    result.ContentTypes.Add(MediaTypeNames.Application.Xml);

                    /*
                      * output -
                      * {
                      *      "Id": "The Id field is required."
                      * }
                      */
                    return result;
                };
            }); 
    }
  ```