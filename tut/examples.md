# Pathy Examples

## Constructing paths

Typically, you want to import both `Path` and all of the helpers in its companion:
```scala
import pathy.Path, Path._
```

To create paths, use either `rootDir or `currentDir`, and then add segments with `</>`, `dir`, and `file`.
```scala

rootDir </> file("afile.txt")

currentDir </> dir("somedir")
```

Note that the resulting type tracks the kind of path:
```scala
val p1: Path[Abs, File, Sandboxed] = rootDir </> file("afile.txt")
val p2: Path[Rel, Dir, Sandboxed] = currentDir </> dir("somedir")
```

The special operator `<::>` builds paths that refer to parent directories. Note that the resulting path is always `Unsandboxed`, meaning you can't be sure that it refers to a file inside the current directory:
```scala
val p3: Path[Rel, Dir, Unsandboxed] = currentDir <::> dir("somedir")
```

If the types don't match, you get a _compile time_ error:
```scala
scala> val impossible: Path[Rel, Dir, Sandboxed] = rootDir </> file("afile.txt")
<console>:15: error: type mismatch;
 found   : pathy.Path[pathy.Path.Abs,pathy.Path.File,Nothing]
 required: pathy.Path[pathy.Path.Rel,pathy.Path.Dir,pathy.Path.Sandboxed]
val impossible: Path[Rel, Dir, Sandboxed] = rootDir </> file("afile.txt")
                                                    ^
```

A `Show` instance is provided, which renders paths as they would be entered in source code:
```scala
import scalaz._, Scalaz._
```
```scala
scala> p1.shows
res4: String = rootDir </> file("afile.txt")
```

Finally, type aliases are defined for the four interesting combinations, and you can often ignore sandboxing:
```scala
scala> val p1a: AbsFile[_] = p1
p1a: pathy.Path.AbsFile[_] = FileIn(Root,FileName(afile.txt))
```

## Parsing paths

A _codec_ provides simple parsing and printing of paths. A couple of simple codecs are provided:

```scala
val p4: Option[Path[Abs, Dir, Unsandboxed]] = posixCodec.parseAbsDir("/")
val p5: Option[Path[Rel, File, Unsandboxed]] = windowsCodec.parseRelFile(".\\My Documents\\bob.bmp")
```

If you try to parse a path of the wrong type, pathy lets you know:
```scala
scala> posixCodec.parseRelFile("/actuallyADir/")
res5: Option[pathy.Path.RelFile[pathy.Path.Unsandboxed]] = None
```

To print a path, it first needs to be "sandboxed", that is, made relative to some reference path:
```scala
scala> for {
     |   p <- posixCodec.parseRelFile("./foo/bar/baz.txt")
     |   p <- sandbox(currentDir </> dir("foo"), p)
     | } yield posixCodec.printPath(p)
res6: Option[String] = Some(./bar/baz.txt)
```


## Working with paths

_TODO: examples for the mundane stuff_

If you know you're dealing with a File path, some additional functions are available:

```scala
val p6 = rootDir </> file("file.txt")
```
```scala
scala> renameFile(p6, name => FileName(name.value.toUpperCase)).shows
res7: String = rootDir </> file("FILE.TXT")

scala> (p1 <:> "jpg").shows
res8: String = rootDir </> file("afile.jpg")
```

A file path _always_ has both a parent direcory and a file name at the end:
```scala
scala> fileParent(p6)
res9: pathy.Path[pathy.Path.Abs,pathy.Path.Dir,Nothing] = Root

scala> fileName(p6)
res10: pathy.Path.FileName = FileName(file.txt)
```

On the other hand, a directory path may or may not actually contain any directory names:
```scala
scala> dirName(currentDir)
res11: Option[pathy.Path.DirName] = None
```
