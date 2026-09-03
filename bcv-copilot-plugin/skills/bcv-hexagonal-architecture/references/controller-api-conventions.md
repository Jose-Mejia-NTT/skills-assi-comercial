# controller-api-conventions

Controller updates must follow existing project style.

Required annotations on new endpoint methods:
1. @Operation
2. @ApiResponses with 200, 400, 500
3. @ObservableOperation

Additional conventions:
1. Reuse current MediaType and ResponseEntity style from existing controllers.
2. Keep endpoint naming and HTTP verb consistent with nearby methods.
3. Reuse existing request/response wrappers if the project already has them.
4. Keep controller thin: delegate orchestration to command/query application services.
