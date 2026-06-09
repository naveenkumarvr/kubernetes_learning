# Kubernetes NetworkPolicy Learning Guide

This README explains the file allowStrictPath.yaml in detail so you can understand exactly how traffic is controlled in this lab.

## Goal of this policy set

The file defines three NetworkPolicy objects that create a strict service path:

1. frontend can call backend on TCP 80
2. backend can call database on TCP 80
3. all three pods can do DNS lookups on port 53 (UDP and TCP)
4. other unwanted paths are denied

In short, the allowed app flow is:

frontend -> backend -> database

## Important NetworkPolicy rules to remember

1. Policies are namespaced. These apply only in namespace policy-lab.
2. Once a pod is selected by a policy for Ingress or Egress, that direction becomes deny-by-default unless explicitly allowed.
3. Multiple policies affecting the same pod are additive allows (union of allowed traffic), not “last one wins”.
4. In a single rule item, from or to plus ports means AND logic.

## File walkthrough by line ranges

Source file: allowStrictPath.yaml

### Document 1: frontend-policy (lines 1 to 27)

Line-by-line meaning:

1. 1: apiVersion: networking.k8s.io/v1
	Uses stable networking API version that includes NetworkPolicy.
2. 2: kind: NetworkPolicy
	Declares this resource type.
3. 3 to 5: metadata block
	Names this policy frontend-policy in namespace policy-lab.
4. 6: spec
	Starts policy behavior definition.
5. 7 to 9: podSelector app=frontend
	Targets frontend pods only.
6. 10 to 12: policyTypes Ingress and Egress
	Both incoming and outgoing traffic for frontend pods are controlled.
7. 13: ingress: []
	No allow rules for ingress, so incoming traffic to frontend is denied.
8. 14: egress
	Starts outgoing allow rules.
9. 15 to 19: DNS egress allow
	Allows frontend to send DNS queries to port 53 over UDP and TCP.
10. 20 to 26: backend egress allow
	Allows frontend to connect only to pods labeled app=backend on TCP 80.
11. 27: ---
	YAML separator. Next policy starts.

Practical effect:

1. frontend cannot be called by other pods
2. frontend can only go out for DNS and backend:80

### Document 2: backend-policy (lines 28 to 61)

Line-by-line meaning:

1. 28 to 32: apiVersion, kind, metadata
	Defines backend-policy in policy-lab.
2. 33 to 36: podSelector app=backend
	Targets backend pods only.
3. 37 to 39: policyTypes Ingress and Egress
	Both directions are controlled for backend pods.
4. 40 to 47: ingress from frontend on TCP 80
	backend accepts traffic only if source pod label is app=frontend and destination port is 80/TCP.
5. 48 to 53: DNS egress allow
	backend may perform DNS lookups on port 53 (UDP and TCP).
6. 54 to 60: egress to database on TCP 80
	backend may call only app=database pods on TCP 80.
7. 61: ---
	YAML separator. Third policy starts.

Practical effect:

1. backend can be reached only by frontend:80
2. backend can go out only to DNS and database:80

### Document 3: database-policy (lines 62 to 87)

Line-by-line meaning:

1. 62 to 66: apiVersion, kind, metadata
	Defines database-policy in policy-lab.
2. 67 to 70: podSelector app=database
	Targets database pods only.
3. 71 to 73: policyTypes Ingress and Egress
	Both directions are controlled for database pods.
4. 74 to 81: ingress from backend on TCP 80
	database accepts traffic only from app=backend pods on TCP 80.
5. 82 to 87: DNS egress allow
	database can do DNS queries on port 53 (UDP and TCP).

Practical effect:

1. database is reachable only from backend:80
2. database cannot call frontend or backend on app ports
3. database can still resolve DNS names

## Why DNS (53 UDP and 53 TCP) is allowed

DNS commonly uses UDP 53, but TCP 53 is also used for larger responses or fallback. Allowing both avoids confusing name resolution failures.

## End-to-end behavior summary

Allowed:

1. frontend -> backend:80
2. backend -> database:80
3. frontend, backend, database -> DNS:53 (UDP and TCP)

Denied examples:

1. frontend -> database:80
2. database -> backend:80
3. any pod -> frontend ingress
4. traffic on non-whitelisted ports

## How to verify this in a cluster

1. Apply app manifests and this policy manifest in namespace policy-lab.
2. Exec into frontend and curl backend service on port 80. It should work.
3. Exec into frontend and curl database service on port 80. It should fail.
4. Exec into backend and curl database service on port 80. It should work.
5. Exec into database and curl backend service on port 80. It should fail.
6. Run DNS lookup from each pod. It should work.

## Common beginner confusion

1. ingress: [] means deny all ingress, not allow all.
2. Omitting egress rules after selecting Egress in policyTypes means deny all egress.
3. podSelector inside from or to matches pod labels in the same namespace unless namespaceSelector is also used.

## Mental model

Think of each selected pod as having two lockable doors:

1. Ingress door (who can enter)
2. Egress door (where it can go)

This file locks both doors for frontend, backend, and database, then opens only specific, minimal paths.
