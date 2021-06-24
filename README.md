# uber-go/gopatch confusing import rewrite behaviour

This repository contains some small example go sources as well as some patches that produce some
confusing behaviour. The expectation is that a patch is applied like:
```
gopatch -p $patch .
```
and then the sources are reset after inspection with a:
```
git checkout .
```

## change-both-imports.patch

`change-both-imports.patch` is intended to follow the best practises documented in the workaround section
of uber-go/gopatch#2. This results in the new import being added twice.
```diff
diff --git a/main-no-comments.go b/main-no-comments.go
index f464237..38ea0ca 100644
--- a/main-no-comments.go
+++ b/main-no-comments.go
@@ -1,8 +1,9 @@
 package main
 
 import (
-	lib "github.com/fake/lib"
 	library "github.com/fake/library"
+	"github.com/totallyreal/lib"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main-unnamed-no-comments.go b/main-unnamed-no-comments.go
index f22129d..30b3220 100644
--- a/main-unnamed-no-comments.go
+++ b/main-unnamed-no-comments.go
@@ -1,8 +1,9 @@
 package main
 
 import (
-	"github.com/fake/lib"
 	"github.com/fake/library"
+	"github.com/totallyreal/lib"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main-unnamed.go b/main-unnamed.go
index 34c6d10..0a564c9 100644
--- a/main-unnamed.go
+++ b/main-unnamed.go
@@ -1,8 +1,9 @@
 package main
 
-import (
-	"github.com/fake/lib" // this comment is attached to github.com/fake/lib
+import ( // this comment is attached to github.com/fake/lib
 	"github.com/fake/library"
+	"github.com/totallyreal/lib"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main.go b/main.go
index c22a1aa..7edf8ce 100644
--- a/main.go
+++ b/main.go
@@ -1,8 +1,9 @@
 package main
 
-import (
-	lib "github.com/fake/lib" // this comment is attached to github.com/fake/lib
+import ( // this comment is attached to github.com/fake/lib
 	library "github.com/fake/library"
+	"github.com/totallyreal/lib"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
```

## change-named-imports.patch

`change-named-imports.patch` is a patch that includes a metavariable matching the imported package.
This works as expected, however, if there is a comment attached to the import being changed
it seems to be promoted to the "next AST node up" or something.

```diff
diff --git a/main-no-comments.go b/main-no-comments.go
index f464237..463f9f3 100644
--- a/main-no-comments.go
+++ b/main-no-comments.go
@@ -1,8 +1,8 @@
 package main
 
 import (
-	lib "github.com/fake/lib"
 	library "github.com/fake/library"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main-unnamed-no-comments.go b/main-unnamed-no-comments.go
index f22129d..474ada5 100644
--- a/main-unnamed-no-comments.go
+++ b/main-unnamed-no-comments.go
@@ -1,8 +1,8 @@
 package main
 
 import (
-	"github.com/fake/lib"
 	"github.com/fake/library"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main-unnamed.go b/main-unnamed.go
index 34c6d10..bd9337b 100644
--- a/main-unnamed.go
+++ b/main-unnamed.go
@@ -1,8 +1,8 @@
 package main
 
-import (
-	"github.com/fake/lib" // this comment is attached to github.com/fake/lib
+import ( // this comment is attached to github.com/fake/lib
 	"github.com/fake/library"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main.go b/main.go
index c22a1aa..3547702 100644
--- a/main.go
+++ b/main.go
@@ -1,8 +1,8 @@
 package main
 
-import (
-	lib "github.com/fake/lib" // this comment is attached to github.com/fake/lib
+import ( // this comment is attached to github.com/fake/lib
 	library "github.com/fake/library"
+	lib "github.com/totallyreal/lib"
 )
 
 func main() {
```

## change-unnamed-imports.patch

`change-unnamed-imports.patch` is a patch that does not include a metavariable for the import that is being
changed. This leads to the new import being added but the old not being removed in both the named and unnamed
cases.

```diff
diff --git a/main-no-comments.go b/main-no-comments.go
index f464237..587a504 100644
--- a/main-no-comments.go
+++ b/main-no-comments.go
@@ -3,6 +3,7 @@ package main
 import (
 	lib "github.com/fake/lib"
 	library "github.com/fake/library"
+	"github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main-unnamed-no-comments.go b/main-unnamed-no-comments.go
index f22129d..6f8bc0d 100644
--- a/main-unnamed-no-comments.go
+++ b/main-unnamed-no-comments.go
@@ -3,6 +3,7 @@ package main
 import (
 	"github.com/fake/lib"
 	"github.com/fake/library"
+	"github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main-unnamed.go b/main-unnamed.go
index 34c6d10..ab5c9cd 100644
--- a/main-unnamed.go
+++ b/main-unnamed.go
@@ -3,6 +3,7 @@ package main
 import (
 	"github.com/fake/lib" // this comment is attached to github.com/fake/lib
 	"github.com/fake/library"
+	"github.com/totallyreal/lib"
 )
 
 func main() {
diff --git a/main.go b/main.go
index c22a1aa..fd8dfda 100644
--- a/main.go
+++ b/main.go
@@ -3,6 +3,7 @@ package main
 import (
 	lib "github.com/fake/lib" // this comment is attached to github.com/fake/lib
 	library "github.com/fake/library"
+	"github.com/totallyreal/lib"
 )
 
 func main() {
```
