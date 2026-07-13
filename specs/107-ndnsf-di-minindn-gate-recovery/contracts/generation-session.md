# Qwen Generation Session Contract

One normal collaboration request establishes candidate, providers, roles,
security, attempt, lease, deadline, and key scopes for one 1-32 token generation.

Each token epoch executes ordered Stage 0, Stage 1, and Stage 2 work. Stage 2
publishes a token feedback object for the next Stage 0 epoch. Dependency names
bind session, attempt, token epoch, producer role/provider boot, plan, artifact,
and payload digest. Provider-local KV state binds the same identities.

The session permits one replacement attempt. Replacement increments attempt
epoch, cancels/supersedes the old attempt, rejects late objects/finals, and
rebuilds from full context when compatible KV is unavailable. It never extends
the original deadline or bypasses any security/lease/digest check.

Exactly one final response contains the complete generated-token bundle,
expected count/digest, provider evidence, and terminal status. Intermediate
token feedback is not a final response.
