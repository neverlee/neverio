title: Go编译器修改
date: 2015-08-04 15:32:41
tags: [Go,变量,包,警告]
categories: 小笔记
description: 将go语言的变量，包未使用错误改成警告（更新至1.5版本的go编译器）
---

Go语言将variable declared but not used和package imported but not used设计成错误，正常使用无可厚非，但调试代码时会非常恼人。下面，就通过修改go源码将这两类错误改为警告。PS：不要害怕，因为编译go编译器只需要一两分钟

* golang1.4及以下版本修改方式
    解决variable declared but not used，通过修改go/src/cmd/gc/walk.c
    ``` c
for(l=fn->dcl; l; l=l->next) {
    if(l->n->op != ONAME || (l->n->class&~PHEAP) != PAUTO || l->n->sym->name[0] == '&' || l->n->used)
        continue;
    if(l->n->defn && l->n->defn->op == OTYPESW) {
        if(l->n->defn->left->used)
            continue;
        lineno = l->n->defn->left->lineno;
        //修改此行为下一行 yyerror("%S declared and not used", l->n->sym);
        warn("[Warning] %S declared and not used", l->n->sym);
        l->n->defn->left->used = 1; // suppress repeats
    } else {
        lineno = l->n->lineno;
        //修改此行为下一行 yyerror("%S declared and not used", l->n->sym);
        warn("[Warning] %S declared and not used", l->n->sym);
    }
}
    ```

    解决package imported but not used，通过修改go/src/cmd/gc/lex.c
    ``` c
for(h=0; h<NHASH; h++) {
    for(s = hash[h]; s != S; s = s->link) {
        if(s->def == N || s->pkg != localpkg)
            continue;
        if(s->def->op == OPACK) {
            // throw away top-level package name leftover
            // from previous file.
            // leave s->block set to cause redeclaration
            // errors if a conflicting top-level name is
            // introduced by a different file.
            if(!s->def->used && !nsyntaxerrors) {
                //修改此行为下面两行 pkgnotused(s->def->lineno, s->def->pkg->path, s->name);
                lineno = s->def->lineno;
                warn("[Warning] imported and not used: \"%Z\"" , s->def->pkg->path);
            }
            s->def = N;
            continue;
        }
    ```

    修改完后，cd到go/src下运行sudo ./make.bash，稍等，搞定！

    PS：得益于go变态的编译速度，强烈建议从源码安装go。

* golang1.5修改方法
    解决variable declared but not used，通过修改go/src/cmd/compile/internal/gc/walk.go中的`func walk(fn *Node)`函数

    ``` go
	for l := fn.Func.Dcl; l != nil; l = l.Next {
		if l.N.Op != ONAME || l.N.Class&^PHEAP != PAUTO || l.N.Sym.Name[0] == '&' || l.N.Used {
			continue
		}
		if defn := l.N.Name.Defn; defn != nil && defn.Op == OTYPESW {
			if defn.Left.Used {
				continue
			}
			lineno = defn.Left.Lineno
			//修改此行为下面这行 Yyerror("%v declared and not used", l.N.Sym)
			Warn("%v declared and not used", l.N.Sym)
			defn.Left.Used = true // suppress repeats
		} else {
			lineno = l.N.Lineno
			//修改此行为下面这行 Yyerror("%v declared and not used", l.N.Sym)
			Warn("%v declared and not used", l.N.Sym)
		}
	}
    ```

    解决package imported but not used，通过修改go/src/cmd/compile/internal/gc/lex.go中的`func mkpackage(pkgname string)`函数
    ``` go
    for _, s := range localpkg.Syms {
        if s.Def == nil {
            continue
        }
        if s.Def.Op == OPACK {
            // throw away top-level package name leftover
            // from previous file.
            // leave s->block set to cause redeclaration
            // errors if a conflicting top-level name is
            // introduced by a different file.
            if !s.Def.Used && nsyntaxerrors == 0 {
                //修改此行为下面两行 pkgnotused(int(s.Def.Lineno), s.Def.Name.Pkg.Path, s.Name)
                lineno = s.Def.Lineno
                Warn("imported and not used: %q", s.Def.Name.Pkg.Path)

            }
            s.Def = nil
            continue
        }
    ```

    golang1.5编译需要golang1.4，设置好GOROOT_BOOTSTRAP环境变量为golang1.4所在目录，cd到golang1.5的源码（go/src）下运行sudo ./make.bash，稍等，搞定！
