# hexagonal-rules

Architecture dependency rules:

Allowed:
1. input -> core
2. output -> core
3. app -> all (wiring only)

Forbidden:
1. core -> input
2. core -> output
3. input -> output

Generation rules:
1. One invocation creates one use case vertical slice.
2. Use case classes are plain Java classes, no Spring stereotype.
3. Ports are interfaces in core.
4. Adapters implement core ports.
5. New use case must be wired in UseCaseConfig.
