# klee-mo

changes to the klee docker code to make it output the memory object files

**`Interpreter.h`**

```cpp
virtual void dumpMM( const ExecutionState &state ) = 0;

virtual void dumpMMStr( const ExecutionState &state, std::unique_ptr<llvm::raw_fd_ostream> &f ) = 0;
```

**`Executor.cpp`**

add `mo->setName(v.getGlobalIdentifier());` below line 776 (`klee_error("out of memory");`)


add the following to the bottom of the file

```cpp
void Executor::dumpMM(const ExecutionState &state) {
  const MemoryMap& mm = state.addressSpace.objects;
  llvm::raw_ostream& os = errs();
  os << "{";
  MemoryMap::iterator it = mm.begin();
  MemoryMap::iterator ie = mm.end();
  if (it!=ie) {
    ObjectState* mobj = it->second.get();
    os << "MO" << it->first->id << " (" << it->first->name << "): ";
    if (mobj->size < 10) {
      mobj->print();
    } else {
      os << "too large to print\n";
    }
    for (++it; it!=ie; ++it) {
      mobj = it->second.get();
      os << ", MO" << it->first->id << " (" << it->first->name << "):";
      if (mobj->size < 10) {
	mobj->print();
      } else {
	os << "too large to print\n";
      }
    }
  }
  os << "}\n";
}

void Executor::dumpMMStr(const ExecutionState &state, std::unique_ptr<llvm::raw_fd_ostream> &f) {
  const MemoryMap& mm = state.addressSpace.objects;
  *f << "{";
  MemoryMap::iterator it = mm.begin();
  MemoryMap::iterator ie = mm.end();
  if (it!=ie) {
    if (it->first->name.compare("unnamed") != 0) {
      ObjectState* mobj = it->second.get();
      *f << "MO" << it->first->id << " (" << it->first->name << "): ";
      if (mobj->size < 10) {
        mobj->filePrint(f);
      } else {
        *f << "too large to print\n";
      }
    }
    for (++it; it!=ie; ++it) {
      if (it->first->name.compare("unnamed") != 0) {
        ObjectState* mobj = it->second.get();
        *f << ", MO" << it->first->id << " (" << it->first->name << "):";
        if (mobj->size < 10) {
          mobj->filePrint(f);
        } else {
          *f << "too large to print\n";
        }
      }
    }
  }
  *f << "}\n";
}
```

**`Executor.h`**

```cpp
virtual void dumpMM( const ExecutionState &state ) override;

virtual void dumpMMStr( const ExecutionState &state, std::unique_ptr<llvm::raw_fd_ostream> &f ) override;
```

**`Memory.cpp`**

```cpp
void ObjectState::filePrint(std::unique_ptr<llvm::raw_fd_ostream> &f) {
  *f << "-- ObjectState --\n";
  *f << "\tMemoryObject ID: " << object->id << "\n";
  *f << "\tRoot Object: " << updates.root << "\n";
  *f << "\tSize: " << size << "\n";

  *f << "\tBytes:\n";
  for (unsigned i=0; i<size; i++) {
    *f << "\t\t["<<i<<"]"
               << " concrete? " << isByteConcrete(i)
               << " known-sym? " << isByteKnownSymbolic(i)
               << " flushed? " << isByteFlushed(i) << " = ";
    ref<Expr> e = read8(i);
    *f << e << "\n";
  }

  *f << "\tUpdates:\n";
  for (const auto *un = updates.head.get(); un; un = un->next.get()) {
    *f << "\t\t[" << un->index << "] = " << un->value << "\n";
  }
}
```

**`Memory.h`**

```cpp
void filePrint(std::unique_ptr<llvm::raw_fd_ostream> &f);
```

**`main.cpp`**
add the following below line 97

```cpp
cl::opt<bool>
WriteMO("write-mo",
        cl::desc("Write .mo files for each test case (default=false)"),
        cl::cat(TestCaseCat));
```

add the following below line 555

```cpp
if (WriteMO) {
  auto f = openTestFile("mo", id);
  if (f) {
    m_interpreter->dumpMMStr(state, f);
  }
}
```


---

Add the option `--write-mo` when running klee, you will see a folder called `mo` that includes the memory object files.